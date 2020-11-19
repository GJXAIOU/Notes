---
style: summer
---

# SSM框架原理,作用及使用方法

## 一、框架原理

### （一）SSM 作用

SSM 框架是 spring MVC ，spring 和 mybatis 框架的整合，是标准的 MVC 模式，将整个系统划分为表现层，controller层，service 层，DAO 层四层；

- 使用spring MVC负责**请求的转发和视图管理**
- spring实现业务对象管理
- mybatis作为数据对象的持久化引擎

### （二）实现原理

- **spring MVC**
  - DNS 负责域名的解析, 比如访问`www.baidu.com` 先通过DNS获取相应的服务器IP和端口；
  - 请求消息到达端口以后由**Tomcat 主动去询问**自己占用的端口是否由请求发来；
  - 如果有请求 Tomcat **交给对应的项目**处理；
  - 客户端发送请求到 DispacherServlet（前端控制器即分发器），这里可以设置拦截器，对请求进行过滤处理；
  - 由 DispacherServlet 控制器查询 HanderMapping，通过解析请求，判断请求希望执行的具体方法，即找到处理请求的 Controller；
这个map表由很多key:value键值对组成, key值是controller的名字(@mapping ...), value值是该controller所在类的地址和方法签名;
(一个类中可能由很多controller)这个找到controller位置并实例化的过程叫做**反射**
反射得到实例后通过**代理**执行相应的方法即相应controller;
  - 通过 HandlerAdapter 调用具体的 Controller 方法；
  - Controller 调用业务逻辑处理后，返回 ModelAndView，即控制器结果返回给视图解析器；
  - DispacherServlet 查询视图解析器，找到ModelAndView 指定的视图
  - 视图负责将结果显示到客户端

![SpringMVC 流程](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/SpringMVC%20%E6%B5%81%E7%A8%8B.png)

**上面补充：**
- 拦截器：拦截器主要是在 SpringMVC 的 controller 处理请求之前，对请求做一些处理，**拦截器拦截的是 Controller**，拦截器分为：HandlerInterceptor（springmvc 的）和 WebRequestInterceptor（spring 的），同时里面一共有三个方法，可以实现部分，主要是拦截位置不同，整体的指向顺序为： preHandle ->进入控制器（controller）-> postHandler -> JSP -> afterCompletion；

- **配置文件**
  - web.xml：在 `<servlet>` 中要配置前端控制器，springmvc 的配置文件位置、自启动、拦截器以及编码过滤器；
  - springmvc.xml ：配置扫描包，即项目中所有 controller 位置，配置注解驱动（一般使用注解），配置不要拦截静态资源、自定义视图解析器（就是前后缀，可以和 controller 的返回值相连接，构成完整的页面路径）；
  - controller 中代码示例：
```java
@controller // 将该类交给容器管理
public class demo(){
    @RequestMapping("A")
    public String hello(){
        return "main";
    }
    @RequestMapping("B")
    public String hello(){
        return "main";
    }
}
```
然后在 jsp 的例如表单中的 action 属性配置为：A，将这个请求和控制器进行映射了，这个请求对应会调用 hello 这个控制器；

- 同时 springmvc 将 jsp 中的参数值传递给 controller 中可以采用方法有：
  - HandlerMenthod；
  - Map
  - Model 接口
  - ModelAndView 类

- Spring：我们平时开发接触最多的估计就是IOC容器，**它可以装载bean（也就是我们Java中的类，当然也包括service dao里面的**），有了这个机制，我们就不用在每次使用这个类的时候为它初始化，很少看到关键字new。另外spring的aop，事务管理等等都是我们经常用到的。

- Mybatis：mybatis是对jdbc的封装，它让数据库底层操作变的透明。mybatis的操作都是围绕一个sqlSessionFactory实例展开的。**mybatis通过配置文件关联到各实体类的Mapper文件，Mapper文件中配置了每个类对数据库所需进行的sql语句映射**。在每次与数据库交互时，通过sqlSessionFactory拿到一个sqlSession，再执行sql命令。

### （三）使用方法
要完成一个功能：

- 先写实体类entity，定义对象的属性，（可以参照数据库中表的字段来设置，数据库的设计应该在所有编码开始之前）。
- 写Mapper.xml（Mybatis），其中定义你的功能，对应要对数据库进行的那些操作，比如 insert、selectAll、selectByKey、delete、update等。
- 写Mapper.java，将**Mapper.xml中的操作按照id映射成Java函数**。
- 写Service.java，为==**控制层提供服务，接受控制层的参数，完成相应的功能，并返回给控制层**==。
- 写Controller.java，**连接页面请求和服务层**，**获取页面请求的参数，通过自动装配，映射不同的URL到相应的处理函数，并获取参数，对参数进行处理，之后传给服务层**。
-  写JSP页面调用，请求哪些参数，需要获取什么数据。

DataBase --> Entity --> Mapper.xml --> Mapper.Java --> Service.java --> Controller.java --> .Jsp

**说明：**

- Spring MVC  拥有控制器，作用跟Struts2 类似，接收外部请求，**解析参数传给服务层**
- Spring 容器属于协调上下文，**管理对象间的依赖，提供事务机制**
- mybatis 属于orm持久层框架，**将业务实体与数据表联合起来**；

- Spring MVC  控制层，想当于 Struts的作用
- Spring 控制反转和依赖注入：  创建对象交由容器管理，达到了解耦的作用
- mybatis 主要用来操作数据库（数据库的增删改查）

- IOC:控制反转，是一种降低对象之间耦合关系的设计思想，面试的时候最好能说出来个例子，加深理解。例子：租房子，以前租房子需要一个房子一个房子找，费时费力，然后现在加入一个房屋中介，把你需要的房型告诉中介，就可以直接选到需要的房子，中介就相当于spring容器。

- AOP:面向切面编程，是面向对象开发的一种补充，**它允许开发人员在不改变原来模型的基础上动态的修改模型以满足新的需求**，如：动态的增加日志、安全或异常处理等。AOP使业务逻辑各部分间的耦合度降低，提高程序可重用性，提高开发效率。

### 持久层（数据访问层）：DAO层（mapper）（Data Access Object）

*   DAO层：==DAO层主要是做数据持久层的工作，负责与数据库进行联络的一些任务都封装在此==，

    *   DAO层的设计首先是设计DAO的接口，
    *   然后在Spring的配置文件中定义此接口的实现类，
    *   然后就可在模块中调用此接口来进行数据业务的处理，而不用关心此接口的具体实现类是哪个类，显得结构非常清晰，
    *   DAO层的数据源配置，以及有关数据库连接的参数都在Spring的配置文件中进行配置。

### 业务逻辑层：Service层

*   Service层：==Service层主要负责业务模块的逻辑应用设计==。
    *   首先设计接口，再设计其实现的类
    *   接着再在Spring的配置文件中配置其实现的关联。这样我们就可以在应用中调用Service接口来进行业务处理。
    *   Service层的业务实现，具体要调用到已定义的DAO层的接口，
    *   封装Service层的业务逻辑有利于通用的业务逻辑的独立性和重复利用性，程序显得非常简洁。

## 表现控制层：Controller层（Handler层）

*   Controller层:==Controller层负责具体的**业务模块流程的控制**==，
    *   在此层里面要调用Service层的接口来控制业务流程，
    *   控制的配置也同样是在Spring的配置文件里面进行，针对具体的业务流程，会有不同的控制器，我们具体的设计过程中可以将流程进行抽象归纳，设计出可以重复利用的子单元流程模块，这样不仅使程序结构变得清晰，也大大减少了代码量。

### View层

*   View层 此层与控制层结合比较紧密，需要二者结合起来协同工发。==View层主要负责前台jsp页面的表示==.

### 各层联系

*   DAO层，Service层这两个层次都可以单独开发，互相的耦合度很低，完全可以独立进行，这样的一种模式在开发大项目的过程中尤其有优势
*   Controller，View层因为耦合度比较高，因而要结合在一起开发，但是也可以看作一个整体独立于前两个层进行开发。这样，在层与层之前我们只需要知道接口的定义，调用接口即可完成所需要的逻辑单元应用，一切显得非常清晰简单。

*   Service逻辑层设计
    *   Service层是建立在DAO层之上的，建立了DAO层后才可以建立Service层，而Service层又是在Controller层之下的，**因而Service层应该既调用DAO层的接口，又要提供接口给Controller层的类来进行调用**，它刚好处于一个中间层的位置。每个模型都有一个Service接口，每个接口分别封装各自的业务处理方法。



# SSM三大框架的运行流程、原理、核心技术详解！

## 一、Spring部分

### （一）Spring的运行流程

*   第一步：加载spring 配置文件`ApplicationContext ac = new ClassPathXmlApplicationContext("beans.xml");`，ApplicationContext接口，它由BeanFactory接口派生而来，因而提供了BeanFactory所有的功能。**配置文件中的bean的信息是被加载在HashMap中的，一个bean通常包括，id，class，property等，bean的id对应HashMap中的key，value呢就是bean**

具体如何加载？源码如下:

```java
if (beanProperty.element("map") != null){  
Map<String, Object> propertiesMap = new HashMap<String, Object>();  
Element propertiesListMap = (Element)beanProperty.elements().get(0);  
Iterator<?> propertiesIterator = propertiesListMap .elements().iterator();  
while (propertiesIterator.hasNext()) {  
	Element vet = (Element) propertiesIterator.next();  
	if(vet.getName().equals("entry")) {  
		String key = vet.attributeValue("key");  
		Iterator<?> valuesIterator = vet.elements()  .iterator();  
		while (valuesIterator.hasNext()) {  
			Element value = (Element) valuesIterator.next();  
			if (value.getName().equals("value")){  
				propertiesMap.put(key, value.getText());  
			}  
			if (value.getName().equals("ref")) {  
				propertiesMap.put(key, new String[]{
						value.attributeValue("bean") 
						});  
			}  
		}  
	}  
}  
bean.getProperties().put(name, propertiesMap);  
} 
```

*   第二步：调用getBean方法，getBean是用来获取applicationContext.xml文件里bean的，（）写的是bean的id。一般情况都会强转成我们对应的业务层（接口）。例如`SpringService springService =(SpringService)ac.getBean("Service");`
*   第三步：这样我们就可以调用业务层(接口实现)的方法。

具体如下：
![原理图](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20180415194047988.png)
Java反射博大精深，我也不很懂，具体请查看[Java基础之—反射](https://blog.csdn.net/sinat_38259539/article/details/71799078)

**那么bean中的东西到底是怎么注入进去的？**简单来讲，就是在实例化一个bean时，实际上就实例化了类，它通过**反射**调用类中set方法将事先保存在HashMap中的类属性注入到类中。这样就回到了我们Java最原始的地方，**对象.方法，对象.属性**

### （二）Spring的原理

*   **什么是spring？**

*   **spring是一个容器框架，它可以接管web层，业务层，dao层，持久层的各个组件，并且可以配置各种bean， 并可以维护bean与bean的关系，当我们需要使用某个bean的时候，我们可以直接getBean(id)，使用即可**

*   Spring目的：就是让对象与对象（模块与模块）之间的关系没有通过代码来关联，都是通过配置类说明管理的（Spring根据这些配置 内部通过反射去动态的组装对象） ，Spring是一个容器，凡是在容器里的对象才会有Spring所提供的这些服务和功能。

*   层次框架图：
    ![这里写图片描述](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20180415151535222.png)
    说明：
*   **web层：** struts充当web层，接管jsp，action，表单，主要体现出mvc的**数据输入**，**数据的处理**，**数据**的显示分离
    *   **model层：** model层在概念上可以理解为包含了**业务层，dao层，持久层**，需要注意的是，一个项目中，不一定每一个层次都有
    *   **持久层：**体现oop，主要解决关系模型和对象模型之间的**阻抗**

### （三）Spring的核心技术

*   IOC
    *   **ioc（inverse of control）控制反转：** 所谓反转就是把创建对象（bean）和维护对象（bean）之间的关系的权利从程序转移到spring的容器（spring-config.xml）
        ![这里写图片描述](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20180415162059434.png)
        ==说明：`<bean></bean>`这对标签元素的作用：**当我们加载spring框架时，spring就会自动创建一个bean对象，并放入内存**相当于我们常规的new一个对象，而`<property></property>`中的value则是实现了“**对象.set方法**”，这里也体现了注入了概念==
*   DI

    *   **di（dependency injection）依赖注入：** 实际上di和ioc是同一个概念，spring的设计者，**认为di更准确的表示spring的核心**
    *   **spring提倡接口编程，在配合di技术就可以达到层与层解耦的目的**，为什么呢？因为层与层之间的关联，由框架帮我们做了，这样代码之间的耦合度降低，代码的复用性提高
    *   接口编程的好处请访问[Java中接口编程的好处以及实现方式的选择？](https://blog.csdn.net/Song_JiangTao/article/details/82389905) 
*   AOP

    *   **aspect oriented programming（面向切面编程）**
    *   核心：**在不增加代码的基础上，还增加新功能**
    *   理解：**面向切面：其实是，把一些公共的“东西”拿出来，比如说，事务，安全，日志，这些方面，如果你用的到，你就引入。** 也就是说：当你需要在执行一个操作（方法）之前想做一些事情（比如，开启事务，记录日志等等），那你就用before，如果想在操作之后做点事情（比如，关闭一些连接等等），那你就用after。其他类似

### （四）spring整体架构图

![spring整体架构图](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20190903142522892.png)

#### 1、core container（核心容器）

核心容器包含了core，beans，context和expression language四个模块

core和beans模块是框架的基础部分，提供IOC和依赖注入特性。这里的基础概念是BeanFactory，它提供对Factory模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离出依赖关系和配置。

*   core模块主要包含了spring框架基本的黑犀牛工具类，spring的其他组建都要用到这个包里的类，core模块是其他组件的基本核心。当然你也可以在自己应用系统中使用这些工具类。

*   beans模块是所有应用都要用到的，它包含访问配置文件，创建和管理bean以及进行ioc，di操作相关的所有类

*   context模块构建与core和beans模块基础之上，提供了一种类似于JNDI注册器的框架式的对象访问方法。context模块继承了beans的特性，为spring核心提供了大量扩展，添加了对国际化（例如资源绑定）、事件传播、资源加载和对context的透明创建的支持。context模块同事也支持j2ee的一些特性，例如EJB，JMX和基础的远程处理，applicationContext接口是context模块的关键。

*   ExpressionLanguage模块提供了强大的表达式语言，用于在运行时查询和操纵对象。他是jsp2.1规范中定义的unifed expression language的扩展。该语言支持设置/获取属性的值，属性的分配，方法的调用 ，访问数组上下文，容器和索引器，逻辑和算数运算符，命名变量以及从spring的ioc容器中根据名称检索对象。它也支持list投影，选择和一般的list聚合

#### 2、Date Access/Integration

Date Access/Integration层包含JDBC，ORM，OXM,JMS和Transaction模块

*   jdbc模块提供了一个jdbc抽象层，他可以消除冗长的jdbc编码和解析数据厂商特有的错误代码。这个模块包含了spring对jdbc数据访问进行封装的所有类。

*   orm模块为流行的对象-关系映射API，如JPA，JDO，Hibernate，iBatis等，提供了一个交互层。利用ORM封装包，可以混合使用所有spring提供的特性进行O/R映射，如前边提到的简单声明性事务管理。spring框架插入了若干个ORM框架 ，从而提供了ORM的对象关系工具，其中包括JDO，hibernate和iBatisSQl Map。所有这些都遵从spring的通用事务和DAO异常层次结构。

*   OXM模块提供了一个对Object/XML映射实现的抽象层，Object/XML映射实现包括JAXB，Castor，XMLBeans，JiBX和XStream。

*   JMS（java massage service）模块主要包含了一些制造和消费消息的特性

*   Transaction模块支持编程和声明性的事务管理，这些事务类必须实现特地的接口。并且对多有的POJO都适用

#### 3、web

web上下文模块建立在应用程序上下文模块之上，为基于web的应用程序提供了上下文。所以，spring框架支持与Jakarta struts的集成。web模块还简化了处理大部分请求以及将请求参数绑定到域对象的工作。web层包含了web，web-servlet，web-Struts 和web-porlet

*   web模块，提供了基础的面向web的集成特性。例如：多文件上传，使用servlet listeners初始化 Ioc容器已经一个面向web的应用上下文。它还包含spring远程支持中的web的相关部分。
*   web-servlet模块web.servlet.jar：该模块包含spring的model-view-controller（mvc）实现。spring的mbc框架使得模型范围内的代码和webforms之间能够清楚地分离出来。并与spring框架的其他特性集成在一起。
*   web-Struts模块，该模块提供了对struts的支持，使得类在spring应用中能够与一个典型的struts web层集成在一起。注意，该支持在spring3.0中已被弃用。
*   web-portlet模块，提供了用于portlet环境和web-servlet模块的MVC的实现。

#### 4、AOP

aop模块提供了一个符合aop联盟标准的面向切面编程的实现，它让你可以定义例如方法拦截器和切点，从而将逻辑代码分开，降低它们之间的耦合性。利用source-level的元数据功能，还可以将各种行为信息合并到你的代码中，这有点像.Net技术中的attribute概念

通过配置管理特性，springAop模块直接将面向界面的编程功能集成到了spring框架中，所以可以很容易地使用spring框架管理的任何对象支持aop，springAop模块为基于spring的应用程序中的对象提供了事务管理服务。通过使用springAop，不用历来EJB组件，就可以将声明性事务管理集成到应用程序中。

*   Aspects模块提供了AspectJ的集成支持。
*   Instrumentation模块提供了class Instrumentation支持和classloader实现，使用可以再特定的应用服务器上使用。

#### 5、Test

test模块支持使用JUnit和TestNG对spring组件进行测试。

* * *

# 二、Spring MVC部分

### （一）Spring MVC的运行流程

*   springMVC框架
    ![这里写图片描述](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20180503140150984.png)
    ==**框架执行流程（面试必问）**==
    *   1、用户发送请求至前端控制器DispatcherServlet
    *   2、DispatcherServlet收到请求调用HandlerMapping处理器映射器。
    *   3、处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
    *   4、DispatcherServlet通过HandlerAdapter处理器适配器调用处理器
    *   5、执行处理器(Controller，也叫后端控制器)。
    *   6、Controller执行完成返回ModelAndView
    *   7、HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet
    *   8、DispatcherServlet将ModelAndView传给ViewReslover视图解析器
    *   9、ViewReslover解析后返回具体View
    *   10、DispatcherServlet对View进行渲染视图（即**将模型数据填充至视图中**）。
    *   11、DispatcherServlet响应用户

### （二）Spring MVC的原理

*   1、什么是SpringMVC？

*   **springmvc是spring框架的一个模块，springmvc和spring无需通过中间整合层进行整合。**

*   springmvc是一个基于mvc的web框架。
    
*   **mvc**

    *   mvc在b/s系统 下的应用：
        ![这里写图片描述](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20180503140124314.png)
    *   前端控制器DispatcherServlet（不需要程序员开发）
        *   作用接收请求，响应结果，相当于转发器，中央处理器。有了DispatcherServlet减少了其它组件之间的耦合度。
    *   处理器映射器HandlerMapping(不需要程序员开发)
        *   作用：根据请求的url查找Handler
    *   处理器适配器HandlerAdapter
        *   作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler
    *   处理器Handler **(需要程序员开发)**
        *   注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
    *   视图解析器View resolver(不需要程序员开发)
        *   作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
    *   视图View **(需要程序员开发)**
        *   View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf…）
*   **struts2与springMVC的区别？**

  1、Struts2是**类级别的拦截**，一个类对应一个request 上下文, SpringMVC是**方法级别的拦截**,一个方法对应一个request上下文,而方法同时又跟一个url对应，,所以说从架构本身上SpringMVC就容易实现restful url,而struts2的架构实现起来要费劲,因为Struts2中Action的一个方法可以对应一个url ,而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。

  2、由上边原因, SpringMVC的**方法之间基本上独立的**,**独享request response数据,**请求数据通过参数获取,处理结果通过ModelMap交回给框架,方法之间不共享变量,而Struts2搞的就比较乱,虽然方法之间也是独立的,但其**所有Action变量是共享的**,这不会影响程序运行,却给我们编码读程序时带来麻烦,每次来了请求就创建一个Action ,**一个Action对象对应一个request 上下文。**

  3、由于Struts2需要针对每个request进行封装,把request , session等servlet生命周期的变量**封装成一个一 个Map** **,供给每个Action使用**,并保证线程安全,所以在原则上,是**比较耗费内存的。**

  4、拦截器实现机制上， Struts2有以自己的**interceptor机制,** SpringMVC用的是**独立的AOP方式**,这样导致Struts2的配置文件量还是比SpringMVC大。

  5、**SpringMVC的入口是servlet ,而Struts2是filter (这里要指出, filter和servlet是不同的。以前认为filter是servlet的一种特殊)**,这就导致 了二者的机制不同,这里就牵涉到servlet和filter的区别了。

  6、**SpringMVC集成了Ajax ,使用非常方便**，只需一个注解@ResponseBody就可以实现，然后直接返回响应文本即可,**而Struts2拦截器集成了Ajax ,在Action中处理时一般必须安装插件或者自己写代码集成进去**,使用起来也相对不方便。

  7、**SpringMVC验证支持JSR303 ,**处理起来相对更加灵活方便,而**Struts2验证比较繁琐,感觉太烦乱。**

  8、**Spring MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高**(当然Struts2也可以通过不同的目录结构和相关配置做到SpringMVC-样的效果,但是需要xml配置的地方不少)。

  9、设计思想上, **Struts2更加符合0OP的编程思想**，SpringMVC就比较谨慎,在servlet上扩展。

  10、**SpringMVC开发效率和性能高于Struts2。**

  11、SpringMVC可以认为已经**100%零配置。**

### （三）Spring MVC的核心技术

*   注解开发（@Controller,@RequestMapping,@ResponseBody。。。。）
    
    *   还有Spring的诸多注解，这两者是不需要整合的~
*   传参，接参（request）
*   基本配置
*   文件上传与下载
    *   Spring MVC中文件上传需要添加Apache Commons FileUpload相关的jar包,
    *   基于该jar, Spring中提供了MultipartResolver实现类: CommonsMultipartResolver.
*   拦截器
* **其实最核心的还是SpringMVC的执行流程，各个点的作用得搞清楚。**

  


# 三、Mybatis 部分

### （一）Mybatis的运行流程

*   **Mybatis运行流程图**：
    ![这里写图片描述](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/20180420191152719.png)

第一步：配置文件mybatis.xml，大体如下，

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 引入外部文件
		resource:引入项目中配置文件
		url:引入网络中或者路径文件
	-->
    <properties resource="jdbc.properties"/>
    <settings>
        <!--<setting name="mapUnderscoreToCamelCase" value="true" />-->
        <setting name="lazyLoadingEnabled" value="true" />
        <setting name="aggressiveLazyLoading"  value="false" />
        <setting name="cacheEnabled" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.nuc.entity"></package>
    </typeAliases>
    <!-- - - - - - - 数据库环境配置- - - - - - - - - -->
    <environments default="environments">
        <environment id="environments">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driverClass}"/>
                <property name="url" value="${jdbc.jdbcUrl}"/>
                <property name="username" value="${jdbc.user}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- - - - - - - -映射文件路径- - - - - - -->
    <mappers>
        <!--自动扫描包下的映射文件，要求：同名，同目录-->
        <package name="com.nuc.mapper" />
    </mappers>
</configuration>
```

第二步：加载我们的xml文件
第三步：创建SqlSessionFactoryBuilder
第四步：创建SqlSessionFactory
第五步：调用openSession()，开启sqlSession
第六步：getMapper()来获取我们的mapper（接口），mapper对应的映射文件，在加载mybatis.xml时就会加载
第七步：使用我们自己的mapper和它对应的xml来完成我们和数据库交互。即增删改查。
第八步：提交session，关闭session。

代码如下：

```java
String resource = "mybatis-config.xml";
SqlSession sqlSession = null;
InputStream inputStream = Resources.getResourceAsStream(resource);//读取mybatis配置文件
//SqlSessionFactoryBuilder这个类的作用就是为了创建SqlSessionFactory的
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
/**
 *  factory.openSession(); //需手动提交事务
 *   factory.openSession(true); //系统自动提交事务
 */
sqlSession = factory.openSession();
CustomerMapper mapper = sqlSession.getMapper(CustomerMapper.class);
//增删改查的操作
sqlSession.commit();//如果没有提交，数据库的数据不会改变
sqlSession.close();
```

需要注意的是，sqlSession也自带一些数据交互的操作

### （二）Mybatis的原理

*   **什么是Mybatis？**

    *   **mybatis专注sql本身**，需要程序员自己编写sql语句，sql修改、优化比较方便。==**mybatis是一个不完全 的ORM框架**==，虽然程序员自己写sql，mybatis 也可以实现映射（**输入映射、输出映射**）。
    *   mybatis是一个**持久层的框架**，是apache下的顶级项目。
    *   mybatis托管到goolecode下，后来托管到github下:[mybatis Github地址](https://github.com/mybatis/mybatis-3/releases)
    *   mybatis让程序将主要精力放在sql上，通过mybatis提供的映射方式，自由灵活生成（半自动化，大部分需要程序员编写sql）满足需要sql语句。
    *   mybatis可以将向 preparedStatement 中的输入参数自动进行输入映射，将查询结果集灵活映射成java对象。（输出映射）
*   **mybatis底层实现**

    *   mybatis底层还是采用**原生jdbc**来对数据库进行操作的，只是通过 SqlSessionFactory，SqlSession Executor,StatementHandler，ParameterHandler,ResultHandler和TypeHandler等几个处理器封装了这些过程
*   **对原生态jdbc程序（单独使用jdbc开发）问题总结：**

    * 数据库连接，使用时创建，不使用就关闭，对数据库进行频繁连接开启和关闭，造成数据库资源的浪费
解决：**使用数据库连接池管理数据库连接**

    * 将sql 语句硬编码到Java代码中，如果sql语句修改，需要对java代码重新编译，不利于系统维护
 解决：**将sql语句设置在xml配置文件中，即使sql变化，也无需重新编译**

    * 向preparedStatement中设置参数，对占位符位置和设置参数值，硬编码到Java文件中，不利于系统维护
 解决：**将sql语句及占位符，参数全部配置在xml文件中**

    * 从resutSet中遍历结果集数据时，存在硬编码，将获取表的字段进行硬编码，不利于系统维护。
解决：**将查询的结果集，自动映射成java对象**

*   **mybatis工作原理**

    *   mybatis通过配置文件创建sqlsessionFactory，sqlsessionFactory根据配置文件，配置文件来源于两个方面:一个是xml，一个是Java中的注解，获取sqlSession。SQLSession包含了执行sql语句的所有方法，可以通过SQLSession直接运行映射的sql语句，完成对数据的增删改查和事物的提交工作，用完之后关闭SQLSession。

### （三）Mybatis的核心技术

*   **Mybatis输入映射**

    *   通过parameterType指定输入参数的类型，类型可以是简单类型、hashmap、pojo的包装类型
*   **Mybatis输出映射**
    *   **一、resultType**
        *   作用:将查询结果按照sql列名pojo属性名一致性映射到pojo中。

        *   **使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。**

        *   如果查询出来的列名和pojo中的属性名全部不一致，则不会创建pojo对象。

        *   只要查询出来的列名和pojo中的属性有一个一致，就会创建pojo对象

        *   **如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。**

    *   **二、resultMap**

        *   使用association和collection完成**一对一和一对多**高级映射（对结果有特殊的映射要求）。
        *   association：
            *   作用：将关联查询信息映射到一个**pojo对象**中。
            *   场合：为了方便查询关联信息可以使用association将关联订单信息映射为用户对象的pojo属性中，比如：查询订单及关联用户信息。
            *   **使用resultType无法将查询结果映射到pojo对象的pojo属性中，根据对结果集查询遍历的需要选择使用resultType还是resultMap。**
        *   collection：
            *   作用：将关联查询信息映射到一个**list集合**中。
            *   场合：为了方便查询遍历关联信息可以使用collection将关联信息映射到list集合中，比如：查询用户权限范围模块及模块下的菜单，可使用collection将模块映射到模块list中，将菜单列表映射到模块对象的菜单list属性中，这样的作的目的也是方便对查询结果集进行遍历查询。**如果使用resultType无法将查询结果映射到list集合中。**
*   Mybatis的动态sql

*   什么是动态sql？

    *   mybatis核心 对sql语句进行灵活操作，通过表达式进行判断，对sql进行灵活拼接、组装。
    *   包括， where ，if，foreach，choose，when，otherwise，set，trim等标签的使用
*   **数据模型分析思路**
*   **1、每张表记录的数据内容**
        *   分模块对每张表记录的内容进行熟悉，相当 于你学习系统 需求（功能）的过程
    *   **2、每张表重要的字段设置**
        *   非空字段、外键字段
*   **3、数据库级别表与表之间的关系**

    *   外键关系
*   **4、表与表之间的业务关系**

    *   在分析表与表之间的业务关系时一定要建立 在某个业务意义基础上去分析。\color{red}{在某个业务意义基础上去分析。}在某个业务意义基础上去分析。




# 框架整合示例

[原文链接](https://blog.csdn.net/weixin_30764137/article/details/96416103)

# 3、SSM整合

 下面主要介绍三大框架的整合，至于环境的搭建以及项目的创建，参看上面的博文。这次整合我分了2个配置文件，分别是spring-mybatis.xml，包含spring和mybatis的配置文件，还有个是spring-mvc的配置文件，此外有2个资源文件：jdbc.propertis和log4j.properties。完整目录结构如下（最后附上源码下载地址）：
 ![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830174644562-871463833.png)

使用框架的版本：
Spring 4.0.2 RELEASE
Spring MVC 4.0.2 RELEASE
MyBatis 3.2.6

## 3.1、Maven引入需要的JAR包

  在pom.xml中引入jar包
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.javen.maven01</groupId>
    <artifactId>maven01</artifactId>
    <packaging>war</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>maven01 Maven Webapp</name>
    <url>http://maven.apache.org</url>

    <properties>  
        <!-- spring版本号 -->  
        <spring.version>4.0.2.RELEASE</spring.version>  
        <!-- mybatis版本号 -->  
        <mybatis.version>3.2.6</mybatis.version>  
        <!-- log4j日志文件管理包版本 -->  
        <slf4j.version>1.7.7</slf4j.version>  
        <log4j.version>1.2.17</log4j.version>  
    </properties> 

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
             <!-- 表示开发的时候引入，发布的时候不会加载此包 -->  
            <scope>test</scope>
        </dependency>
        <!-- <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency> -->

         <!-- =======spring核心包 ========= -->  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-core</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  

        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-web</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-oxm</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-tx</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  

        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-jdbc</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  

        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-webmvc</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-aop</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  

        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-context-support</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  

        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-test</artifactId>  
            <version>${spring.version}</version>  
        </dependency>  


        <!-- =========  mybatis核心包  ================= -->  
        <dependency>  
            <groupId>org.mybatis</groupId>  
            <artifactId>mybatis</artifactId>  
            <version>${mybatis.version}</version>  
        </dependency>  
         <!-- mybatis/spring包 -->  
        <dependency>  
            <groupId>org.mybatis</groupId>  
            <artifactId>mybatis-spring</artifactId>  
            <version>1.2.2</version>  
        </dependency>  

         <!-- 导入java ee jar 包 -->  
        <dependency>  
            <groupId>javax</groupId>  
            <artifactId>javaee-api</artifactId>  
            <version>7.0</version>  
        </dependency>  

         <!-- 导入Mysql数据库链接jar包 -->  
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <version>5.1.36</version>  
        </dependency>  
        <!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->  
        <dependency>  
            <groupId>commons-dbcp</groupId>  
            <artifactId>commons-dbcp</artifactId>  
            <version>1.2.2</version>  
        </dependency>  

        <!-- JSTL标签类 -->  
        <dependency>  
            <groupId>jstl</groupId>  
            <artifactId>jstl</artifactId>  
            <version>1.2</version>  
        </dependency>  
        <!-- 日志文件管理包 -->  
        <!-- log start -->  
        <dependency>  
            <groupId>log4j</groupId>  
            <artifactId>log4j</artifactId>  
            <version>${log4j.version}</version>  
        </dependency>  

        <!-- 格式化对象，方便输出日志 -->  
        <dependency>  
            <groupId>com.alibaba</groupId>  
            <artifactId>fastjson</artifactId>  
            <version>1.1.41</version>  
        </dependency>  

        <dependency>  
            <groupId>org.slf4j</groupId>  
            <artifactId>slf4j-api</artifactId>  
            <version>${slf4j.version}</version>  
        </dependency>  

        <dependency>  
            <groupId>org.slf4j</groupId>  
            <artifactId>slf4j-log4j12</artifactId>  
            <version>${slf4j.version}</version>  
        </dependency>  
        <!-- log end -->  
        <!-- 映入JSON -->  
        <dependency>  
            <groupId>org.codehaus.jackson</groupId>  
            <artifactId>jackson-mapper-asl</artifactId>  
            <version>1.9.13</version>  
        </dependency>  
        
        <!-- ====上传组件包======== -->  
        <dependency>  
            <groupId>commons-fileupload</groupId>  
            <artifactId>commons-fileupload</artifactId>  
            <version>1.3.1</version>  
        </dependency>  
        <dependency>  
            <groupId>commons-io</groupId>  
            <artifactId>commons-io</artifactId>  
            <version>2.4</version>  
        </dependency>  
        <dependency>  
            <groupId>commons-codec</groupId>  
            <artifactId>commons-codec</artifactId>  
            <version>1.9</version>  
        </dependency>  

    </dependencies>

    <build>
        <finalName>maven01</finalName>
        <plugins>
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>9.2.8.v20150217</version>
                <configuration>
                    <httpConnector>
                        <port>80</port>
                    </httpConnector>
                    <stopKey>shutdown</stopKey>
                    <stopPort>9966</stopPort>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



## 3.2、整合SpringMVC

### 3.2.1、配置spring-mvc.xml

配置里面的注释也很详细，主要是**自动扫描控制器，视图模式，注解的启动**这三个。
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc" xsi:schemaLocation="http://www.springframework.org/schema/beans    
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd    
                        http://www.springframework.org/schema/context    
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd    
                        http://www.springframework.org/schema/mvc    
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">  
    <!-- 自动扫描该包，使SpringMVC认为包下用了@controller注解的类是控制器 -->  
    <context:component-scan base-package="com.javen.controller" />  
    <!-- 扩充了注解驱动，可以将请求参数绑定到控制器参数 -->
    <mvc:annotation-driven/>
    <!-- 静态资源处理  css js imgs -->
    <mvc:resources location="/resources/**" mapping="/resources"/>

    <!--避免IE执行AJAX时，返回JSON出现下载文件 -->  
    <bean id="mappingJacksonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">  
        <property name="supportedMediaTypes">  
            <list>  
                <value>text/html;charset=UTF-8</value>  
            </list>  
        </property>  
    </bean>  
    <!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->  
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">  
        <property name="messageConverters">  
            <list>  
                <ref bean="mappingJacksonHttpMessageConverter" /> <!-- JSON转换器 -->  
            </list>  
        </property>  
    </bean>  

    <!-- 配置文件上传，如果没有使用文件上传可以不用配置，当然如果不配，那么配置文件中也不必引入上传组件包 -->  
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">    
        <!-- 默认编码 -->  
        <property name="defaultEncoding" value="utf-8" />    
        <!-- 文件大小最大值 -->  
        <property name="maxUploadSize" value="10485760000" />    
        <!-- 内存中的最大值 -->  
        <property name="maxInMemorySize" value="40960" />    
        <!-- 启用是为了推迟文件解析，以便捕获文件大小异常 -->
        <property name="resolveLazily" value="true"/>
    </bean>   

    <!-- 配置ViewResolver 。可用多个ViewResolver 。使用order属性排序。   InternalResourceViewResolver 放在最后-->
    <bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
    <property name="order" value="1"></property>
        <property name="mediaTypes">
            <map>
                <!-- 告诉视图解析器，返回的类型为json格式 -->
                <entry key="json" value="application/json" />
                <entry key="xml" value="application/xml" />
                <entry key="htm" value="text/htm" />
            </map>
        </property>
        <property name="defaultViews">
            <list>
                <!-- ModelAndView里的数据变成JSON -->
                <bean class="org.springframework.web.servlet.view.json.MappingJacksonJsonView" />
            </list>
        </property>
        <property name="ignoreAcceptHeader" value="true"></property>
    </bean>

   <!-- 定义跳转的文件的前后缀 ，视图模式配置-->  
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
        <!-- 这里的配置我的理解是自动给后面action的方法return的字符串加上前缀和后缀，变成一个 可用的url地址 -->  
        <property name="prefix" value="/WEB-INF/jsp/" />  
        <property name="suffix" value=".jsp" />  
    </bean>  
</beans>  
```



### 3.2.2、配置web.xml文件

 配置的spring-mvc的Servlet就是为了完成SpringMVC+MAVEN的整合。

web.xml  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">  
    <display-name>Archetype Created Web Application</display-name>  
    <!-- Spring和mybatis的配置文件 -->  
   <!-- <context-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:spring-mybatis.xml</param-value>  
    </context-param> -->
    <!-- 编码过滤器 -->  
    <filter>  
        <filter-name>encodingFilter</filter-name>  
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
        <async-supported>true</async-supported>  
        <init-param>  
            <param-name>encoding</param-name>  
            <param-value>UTF-8</param-value>  
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>encodingFilter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
    <!-- Spring监听器 -->  
   <!-- <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener> -->
    <!-- 防止Spring内存溢出监听器 -->  
    <!-- <listener>  
        <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>  
    </listener> --> 

    <!-- Spring MVC servlet -->  
    <servlet>  
        <servlet-name>SpringMVC</servlet-name>  
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
        <init-param>  
            <param-name>contextConfigLocation</param-name>  
            <param-value>classpath:spring-mvc.xml</param-value>  
        </init-param>  
        <load-on-startup>1</load-on-startup>  
        <async-supported>true</async-supported>  
    </servlet>  
    <servlet-mapping>  
        <servlet-name>SpringMVC</servlet-name>  
        <!-- 此处可以可以配置成*.do，对应struts的后缀习惯 -->  
        <url-pattern>/</url-pattern>  
    </servlet-mapping>  
    <welcome-file-list>  
        <welcome-file>/index.jsp</welcome-file>  
    </welcome-file-list>  

</web-app>  
```

### 3.2.3、Log4j的配置

   为了方便调试，一般都会使用日志来输出信息，Log4j是Apache的一个开放源代码项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。

 Log4j的配置很简单，而且也是通用的，下面给出一个基本的配置，换到其他项目中也无需做多大的调整，如果想做调整或者想了解Log4j的各种配置，参看我转载的一篇博文，很详细：[http://blog.csdn.net/zhshulin/article/details/37937365](http://blog.csdn.net/zhshulin/article/details/37937365)

下面给出配置文件目录：

 ![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830161022859-271175533.png)

log4j.properties
```properties
log4j.rootLogger=INFO,Console,File  
#定义日志输出目的地为控制台  
log4j.appender.Console=org.apache.log4j.ConsoleAppender  
log4j.appender.Console.Target=System.out  
#可以灵活地指定日志输出格式，下面一行是指定具体的格式  
log4j.appender.Console.layout = org.apache.log4j.PatternLayout  
log4j.appender.Console.layout.ConversionPattern=[%c] - %m%n  

#文件大小到达指定尺寸的时候产生一个新的文件  
log4j.appender.File = org.apache.log4j.RollingFileAppender  
#指定输出目录  
log4j.appender.File.File = logs/ssm.log  
#定义文件最大大小  
log4j.appender.File.MaxFileSize = 10MB  
# 输出所以日志，如果换成DEBUG表示输出DEBUG以上级别日志  
log4j.appender.File.Threshold = ALL  
log4j.appender.File.layout = org.apache.log4j.PatternLayout  
log4j.appender.File.layout.ConversionPattern =[%p] [%d{yyyy-MM-dd HH\:mm\:ss}][%c]%m%n  
```



### 3.2.4、使用Jetty测试

![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830165405344-862946687.png)
实体类 User.java

```java
package com.javen.model;

public class User {
    private Integer id;

    private String userName;

    private String password;

    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName == null ? null : userName.trim();
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password == null ? null : password.trim();
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User [id=" + id + ", userName=" + userName + ", password="
                + password + ", age=" + age + "]";
    }
    
    
}
```

UserController.java
```java
package com.javen.controller;
import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestMethod;
import com.javen.model.User;
  
  
@Controller  
@RequestMapping("/user")  
// /user/**
public class UserController {  
    private static Logger log=LoggerFactory.getLogger(UserController.class);
      
    
    // /user/test?id=1
    @RequestMapping(value="/test",method=RequestMethod.GET)  
    public String test(HttpServletRequest request,Model model){  
        int userId = Integer.parseInt(request.getParameter("id"));  
        System.out.println("userId:"+userId);
        User user=null;
        if (userId==1) {
             user = new User();  
             user.setAge(11);
             user.setId(1);
             user.setPassword("123");
             user.setUserName("javen");
        }
       
        log.debug(user.toString());
        model.addAttribute("user", user);  
        return "index";  
    }  
}  
```


 ![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830165700734-984268491.png) ![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830165755156-107359445.png)

在浏览器中输入：http://localhost/user/test?id=1

![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830165857500-208786509.png)

**到此 SpringMVC+Maven 整合完毕**

### 3.3 Spring与MyBatis的整合

取消 3.2.2 web.xml中注释的代码

### 3.3.1、建立JDBC属性文件

jdbc.properties（文件编码修改为utf-8）
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/maven
username=root
password=root
#定义初始连接数  
initialSize=0 #定义最大连接数  
maxActive=20 #定义最大空闲  
maxIdle=20 #定义最小空闲  
minIdle=1 #定义最长等待时间  
maxWait=60000  
```

**此时的目录结构为**

 ![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830170643640-637516889.png)

### 3.3.2、建立spring-mybatis.xml配置文件

这个文件就是用来完成spring和mybatis的整合的。这里面也没多少行配置，主要的就是自动扫描，自动注入，配置数据库。注释也很详细，大家看看就明白了。

spring-mybatis.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc" xsi:schemaLocation="http://www.springframework.org/schema/beans    
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd    
                        http://www.springframework.org/schema/context    
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd    
                        http://www.springframework.org/schema/mvc    
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">  
    <!-- 自动扫描 -->  
    <context:component-scan base-package="com.javen" />  

    <!-- 引入配置文件 -->  
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
        <property name="location" value="classpath:jdbc.properties" />  
    </bean>  

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">  
        <property name="driverClassName" value="${driver}" />  
        <property name="url" value="${url}" />  
        <property name="username" value="${username}" />  
        <property name="password" value="${password}" />  
        <!-- 初始化连接大小 -->  
        <property name="initialSize" value="${initialSize}"></property>  
        <!-- 连接池最大数量 -->  
        <property name="maxActive" value="${maxActive}"></property>  
        <!-- 连接池最大空闲 -->  
        <property name="maxIdle" value="${maxIdle}"></property>  
        <!-- 连接池最小空闲 -->  
        <property name="minIdle" value="${minIdle}"></property>  
        <!-- 获取连接最大等待时间 -->  
        <property name="maxWait" value="${maxWait}"></property>  
    </bean>  

    <!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->  
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
        <property name="dataSource" ref="dataSource" />  
        <!-- 自动扫描mapping.xml文件 -->  
        <property name="mapperLocations" value="classpath:com/javen/mapping/*.xml"></property>  
    </bean>  

    <!-- DAO接口所在包名，Spring会自动查找其下的类 -->  
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
        <property name="basePackage" value="com.javen.dao" />  
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>  
    </bean>  

    <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->  
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
        <property name="dataSource" ref="dataSource" />  
    </bean>  

</beans>  

```


### 3.4、JUnit测试

  经过以上步骤，我们已经完成了Spring和mybatis的整合，这样我们就可以编写一段测试代码来试试是否成功了。

#### 3.4.1、创建测试用表

既然我们需要测试，那么我们就需要建立在数据库中建立一个测试表，这个表建的很简单，SQL语句为：
```sql
-- -- -- 
-- Table structure for `user_t` 
-- ------
DROP TABLE IF EXISTS `user_t`; CREATE TABLE `user_t` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(40) NOT NULL,
  `password` varchar(255) NOT NULL,
  `age` int(4) NOT NULL, PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8; 

-- ---------------------------- 
-- Records of user_t -- 
----------------------------

INSERT INTO `user_t` VALUES ('1', '测试', '345', '24'); INSERT INTO `user_t` VALUES ('2', 'javen', '123', '10');
```



#### 3.4.2、利用MyBatis Generator自动创建代码

参考博文：[http://blog.csdn.net/zhshulin/article/details/23912615](http://blog.csdn.net/zhshulin/article/details/23912615)

 这个可根据表自动创建实体类、MyBatis映射文件以及DAO接口，当然，我习惯将生成的接口名改为IUserDao，而不是直接用它生成的UserMapper。如果不想麻烦就可以不改。完成后将文件复制到工程中。如图：

![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830172815890-478987509.png)

#### 3.4.3、建立Service接口和实现类

![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830173103875-2026469790.png)

下面给出具体的内容：

IUserService.java
```java
package com.javen.service;  

import com.javen.model.User;
  
  
public interface IUserService {  
    public User getUserById(int userId);  
}  
UserServiceImpl.java

package com.javen.service.impl;
import javax.annotation.Resource;  

import org.springframework.stereotype.Service;  
import com.javen.dao.IUserDao;
import com.javen.model.User;
import com.javen.service.IUserService;
  
  
@Service("userService")  
public class UserServiceImpl implements IUserService {  
    @Resource  
    private IUserDao userDao;  
    
    public User getUserById(int userId) {  
        // TODO Auto-generated method stub  
        return this.userDao.selectByPrimaryKey(userId);  
    }  
  
}  
```

#### 3.4.4、建立测试类

 测试类在src/test/java中建立，下面测试类中注释掉的部分是不使用Spring时，一般情况下的一种测试方法；如果使用了Spring那么就可以使用注解的方式来引入配置文件和类，然后再将service接口对象注入，就可以进行测试了。

如果测试成功，表示Spring和Mybatis已经整合成功了。输出信息使用的是Log4j打印到控制台。

```java
package com.javen.testmybatis;

import javax.annotation.Resource;  

import org.apache.log4j.Logger;  
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;  
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;  
import com.alibaba.fastjson.JSON;  
import com.javen.model.User;
import com.javen.service.IUserService;
  
@RunWith(SpringJUnit4ClassRunner.class)     //表示继承了SpringJUnit4ClassRunner类  
@ContextConfiguration(locations = {"classpath:spring-mybatis.xml"})  
  
public class TestMyBatis {  
    private static Logger logger = Logger.getLogger(TestMyBatis.class);  
//  private ApplicationContext ac = null;  
    @Resource  
    private IUserService userService = null;  
  
//  @Before  
//  public void before() {  
//      ac = new ClassPathXmlApplicationContext("applicationContext.xml");  
//      userService = (IUserService) ac.getBean("userService");  
//  }  
  
    @Test  
    public void test1() {  
        User user = userService.getUserById(1);  
        // System.out.println(user.getUserName());  
        // logger.info("值："+user.getUserName());  
        logger.info(JSON.toJSONString(user));  
    }  
}  
```


测试结果 

![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830173453859-1226139271.png)

#### 3.4.5、建立UserController类

UserController.java  控制器   
```java
package com.javen.testmybatis;

import javax.annotation.Resource;  

import org.apache.log4j.Logger;  
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;  
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;  
import com.alibaba.fastjson.JSON;  
import com.javen.model.User;
import com.javen.service.IUserService;
  
@RunWith(SpringJUnit4ClassRunner.class)     //表示继承了SpringJUnit4ClassRunner类  
@ContextConfiguration(locations = {"classpath:spring-mybatis.xml"})  
  
public class TestMyBatis {  
    private static Logger logger = Logger.getLogger(TestMyBatis.class);  
//  private ApplicationContext ac = null;  
    @Resource  
    private IUserService userService = null;  
  
//  @Before  
//  public void before() {  
//      ac = new ClassPathXmlApplicationContext("applicationContext.xml");  
//      userService = (IUserService) ac.getBean("userService");  
//  }  
  
    @Test  
    public void test1() {  
        User user = userService.getUserById(1);  
        // System.out.println(user.getUserName());  
        // logger.info("值："+user.getUserName());  
        logger.info(JSON.toJSONString(user));  
    }  
}  
```


#### 3.4.6、新建jsp页面

file.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Insert title here</title>
</head>
<body>
    <h1>上传文件</h1>
    <form method="post" action="/user/doUpload" enctype="multipart/form-data">
        <input type="file" name="file"/>
        <input type="submit" value="上传文件"/>

    </form>
</body>
</html>

index.jsp

<html>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

showUser.jsp

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">  
<html>  
  <head>  
    <title>测试</title>  
  </head>  

  <body> ${user.userName} </body>  
</html>
```


**至此，完成Spring+SpingMVC+mybatis这三大框架整合完成。**

#### 3.4.7、部署项目

**输入地址：http://localhost/user/jsontype/2**

![](%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88%EF%BC%9ASSM%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86,%E4%BD%9C%E7%94%A8%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.resource/441423-20150830174520609-1121228290.png)

  项目下载地址：[https://github.com/Javen205/SSM](https://github.com/Javen205/SSM)



