

### 我们先说说Spring Boot和SSM本质上的区别

SSM是什么？是三个臭皮匠(裨将)，Spring IoC、Spring MVC、Mybatis的组合。SSM限定死了你只能开发Java Web应用，而且MVC框架必须用Spring MVC，持久层必须用Mybatis，无他！我说的是SSM包含这些啊，没说你不能在这三个基础上自己加其他框架和库上去。

Spring Boot呢？诸葛亮。有了诸葛亮，你用兵的可选方案更多，不管用哪几员将军，出师更顺利。Spring Boot没有和任何MVC框架绑定！没有和任何持久层框架绑定！没有和任何其他业务领域的框架绑定！

你开发Web应用可以用Spring Boot。用spring-boot-starter-web就帮你配置好了Spring MVC。你不想用Spring MVC了，换成Spring WebFLux(用spring-boot-starter-webflux)写响应式Web应用可以吗？当然可以，而且这个是Spring 5主推的新Web框架。

你不开发Web应用，只实现纯粹的数据层业务，开发Spring Cloud Stream和Task也可以。

数据持久层，你可以用Spring Data项目下的任何子项目(JPA\JDBC\MongoDB\Redis\LDAP\Cassandra\Couchbase\Noe4J\Hadoop\Elasticsearch....)，当然用非Spring官方支持的Mybatis也可以。只要用上对应技术或框架的spring-boot-starter-xxx就可以了。

**但是必须要知道，Spring Boot提供的只是这些starters，这些Starter依赖了(maven dependence)对应的框架或技术，但不包含对应的技术或框架本身！**

这就是很多人用“全家桶”这个词来比喻Spring Boot的错误之处。肯德基麦当劳的全家桶里面包含了鸡腿、鸡翅、鸡块，这些东西都是**包含**在里面的，而且是不可选择的。你吃的完是这些，吃不完也是这些。你喜欢吃其中几样，也可能不喜欢吃其中几样。但是Spring Boot不是啊，Spring Boot没有包含Spring MVC，没有包含Mybatis，只有他们对应的starters。

一个更恰当的比喻是，Spring MVC、Spring Data、Websocket这东西对应电脑硬件的显卡、声卡、硬盘、网卡。Spring Boot提供的Starters对应这些硬件的驱动。只要你在主板上插上了这些硬件，Spring Boot提供的对应驱动就能让能让你享受到即插即用(Plug & Play)的体验。Spring Boot提供的是驱动，没有包含显卡、声卡这些硬件本身，这些驱动能够让你DIY的电脑顺畅的引导(boot)并运行起来。

很多Java服务器端的常见第三方框架，Spring Boot都能用Convention over Configuration的方式帮你默认配置好。

具体支持什么看这里：[Spring Boot Reference Guidedocs.spring.io](https://link.zhihu.com/?target=https%3A//docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/%23using-boot-starter)上面官方文档的列表只给出了Spring Boot官方提供的starters，其他第三方自己提供(例如Mybatis的starter)的没有包含在这个列表里。

前几天https://start.spring.io/ 改版了，也可以去看看：

![img](https://pic4.zhimg.com/80/v2-3cb38c1edffe71e6484dadb5338332b9_720w.jpg?source=1940ef5c)

红框里的Dependency是空的，没有限制你必须用哪一个，Web都是可选的。而且你输入Web，会有Spring MVC和Spring WebFlux让你选。

![img](https://pic4.zhimg.com/80/v2-e368efe140bf5defe19f044148d8696b_720w.jpg?source=1940ef5c)


上图下拉列表中，第一个Web就是Spring MVC，第二个Reactive Web就是Spring WebFLux。本文后面我们还会讲到。

关于http://start.spring.io，也就是Spring Initializr，看我回答的另外一个问题，非常简单：

https://www.zhihu.com/question/49941226/answer/619799264


其他答案提到的：配置方便(也就是我上面说的Convention over Configuration)只是表象，其实CoC一直都是Spring贯彻的理念，只是Spring Boot又把它提高到了一个新的高度。

Spring Boot的最核心理念就体现在名字上，Spring就不说啦，那Boot是啥意思啊？
Boot是引导启动的意思，Windows安装启动的过程不就需要引导盘(boot disk或bootable usb)吗？

类似的，前端有一个Bootstrap，也是这个意思，让你快速搭建前端UI。上面提到的那个Spring Initializr，打开页面，看左上角，写的什么？Spring Initializr Bootstrap your application。




如果一家公司一直用SSH/SSM，用了若干年了，经历了大大小小几个项目，基本上配置都已经好了，早就形成了自己的一套默认配置。国内大多数中小公司，新项目的启动就是把上一个项目Copy过来改！还需要用啥Spring Boot？在这种情况下，三个臭皮匠就顶上诸葛亮了！

但是如果你的新项目需要用到以前没整合过的技术呢？比如数据库要用MongoDB了，也许还用到Redis，可能还需要用Spring Security，估计还要上Kafka消息队列，还可能要用Websockets。你的项目怎么才能平稳、顺利、快速启动？手动配置这些东西可能就要花不少时间，而且还不一定配置的好。

明白我上面说的那些，很多问题都没必要问了。例如我回答过的这些问题：

https://www.zhihu.com/question/303235503/answer/537538561

https://www.zhihu.com/question/64671972/answer/568318031

https://www.zhihu.com/question/314112286/answer/613357345


这个问题下原来赞数最多回答推荐的那本书我没看过，但是之前就知道这本书。我看到副标题--Java EE开发的颠覆者，就误导了入门读者，传递了错误观念，仅仅是为了自己的销量。

这个问题下有人不是比喻Spring Boot和SSM之间是自动挡和手动挡的关系吗？有点像。但我还是更喜欢我在这个回答里的比喻：

https://www.zhihu.com/question/49649311/answer/364216794

从马车到汽车是交通出行的颠覆，从燃油车到纯电动车是能源利用的颠覆，从人工驾驶到AI智是驾驶方式的颠覆。但，只是给传统汽车改成自动挡、加装一键启停、无钥匙进入、自动跟车、车道偏离预警，我觉得不算颠覆。

**这个”颠覆“给人的感觉就是：Spring MVC不用学了，Spring其他都不用学了，Spring Boot颠覆了他们的，取代他们的，包含了他们以前所有的功能。然后就误导出了上面那些问题。**

我们再看看这个问题下其他答案
**回答1：两个一样的，只不过是springboot省去了很多配置**
怎么可能一样呢？如果一样，大部分公司都没必要迁移到Spring Boot。因为上面说了，只要做了几个项目，基本也就把SSM配置的差不多了。每个项目都Copy套用就可以了。

**回答2：Spring Boot 就像一个脚手架一样 能让你快速的搭建项目 他不是替代SSM的 至于返回什么 完全看前端需求和文档规定吧**
Spring Boot就像一个脚手架，但绝对不是一个脚手架。什么是脚手架？我们都见过，就是建筑工地盖楼房的时候外面那一层钢管搭建的架子，还有一层绿网，就是方便构建楼房。但是楼房竣工以后，脚手架是要被拆掉的，不会作为物业的一部分交给业主。软件开发中的脚手架也是类似的，帮助快速搭建项目，而脚手架不会作为最终交付成果的一部分。
你用了Spring Boot，那么Spring Boot以及其他starter的jar都会最终进入你打包编译的jar里，作为你成果的一部分。


那么Spring体系当中，有没有真正的脚手架呢？有的，就是答案上面截图的http://start.spring.io，它叫做Spring Initializr Bootstrap your application，**就是通过页面上的操作，很快帮你生成搭建好一个初始化好的Spring Boot应用。但是Spring Initializr自己不会进入你最后打包的jar，Spring Initializr是Spring Boot应用的一个简单脚手架工具。**

怎么又是一个“简单”的脚手架工具了？因为还有Spring Boot Cli啊。Cli是Command Line Interface的缩写，就是命令行工具。Spring Boot Cli通过命令行交互的方式提供了脚手架功能，你可以用它初始化一个Spring Boot项目，也可以用它完成打包jar的工作。

> The init command lets you create a new project by using start.spring.io without leaving the shell
> --Spring Boot 文档
>



例如运行spring init：

> $ spring init --dependencies=web,data-jpa my-project
> Using service at https://start.spring.io
> Project extracted to '/Users/developer/example/my-project'
>

就完成了在http://start.spring.io中创建项目的工作。

Spring Boot不是替代SSM的，这个是对的。因为答案上面解释过了，Spring Boot没有和任何Web MVC绑定，没有和任何数据持久化绑定。Spring Boot自己根本无法完成SSM能完成的工作，需要其他starter作为桥梁，自动帮你配置好对应的框架才行。

**回答3：springboot采用约定大于配置的方式，简化了大量的xml配置，真正做到了开箱即用。减少了web开发的难度**
Spring Boot是采用约定大于配置(就是Convention over Configuration)，简化了大量的XML配置。难道在Spring Boot出来以前Spring Framework或者其他Spring体系下的框架就没有采用CoC吗？就没有提倡用Java Annotation来简化配置吗？ CoC一直就是Spring所倡导的，只是Spring Boot更进一步发扬光大了！

真正做到开箱即用？没有吧，那还需要Spring Initializr干嘛？或者还需要手动配置POM干嘛？还是要做一定定制化的。如果论开箱即用(Out of Box)，我答案里说的传统公司的流程才是开箱即用。新项目Copy老项目就算开箱了，直接上去改就算开始用了。

**回答4：SSM：面相XML编程。SpringBoot：面相注解编程**
Spring Boot出现以前，Spring Framework已经在推荐Java Annotation的配置方式了，在所有的文档里都会同时介绍注解和XML两种配置方式，并且优先介绍注解的方式。

**回答5：ssm是自己买家具装修。spring boot是全屋定制**
SSM不是自己买家具，而是住酒店，酒店房间里的东西就这些，没得换。你只能自己带一些小件日用品。SSM这家酒店你不喜欢，那就换SSH那家去。Spring Boot是全屋定制，这个形容不错。而且家具种类齐全，品种多样。你只要下单就可以，包送货、包安装调试。

**回答6：没有区别，springboot 只是提供了一套默认配置，用于原型的快速开发。**
怎么可能没区别，就不多说了。用于原型快速开发这句话存在严重片面理解。快速开发没问题，但绝对不是只限于开发原型。

Spring Boot文档开章明义就讲了：

> Spring Boot makes it easy to create stand-alone, production-grade Spring-based Applications that you can run.

怎么是Production-grade(产品级)呢？

Spring Boot文档第V章，专门讲了这些，指的是Actuator，用来监控和控制Spring Boot应用。包括Loggers，Metrics，Auditing，HTTP Tracing，Process Monitoring等等产品环境下需要的一套机制。

**回答7：区别就是配置方式不一样，对于实现业务逻辑来说没有任何区别。**
区别不仅仅是配置方式不一样，实现业务逻辑更是大有不同！用SSM开发应用，大多数公司的技术栈还在使用JSP，很少采用前后端分离的。而Spring Boot倡导前后端分离的开发，Spring Boot的应用只向前端提供RESTful API。下面一节会重点说这个。

不再逐一分析了。为什么要批判这些回答？因为这些回答代表的观念不仅仅出现在这个问题下的回答里，其他Spring Boot相关的问题下面都能见到这些观点层出不穷。

### 我们再说说题主在问题描述中表达的困惑

原来使用ssm开发，后来公司用springboot，虽然一直在用但除了springboot简化构建项目以外，还有个别的几个不一样的注解，其他的都感觉差不多，有人看到一个框架说:他虽然使用spring boot构建了项目，但是里面开发还是用的ssm，因为他使用了ModelAndView，那么问题来了，那spring boot中又是返回什么呢？
问题描述中的这段话充分体现了软件开发上的一些根本性的问题。

软件开发，首先是思想行为，其次才是编码。想明白想通了，处处顺风顺水。想不通，觉得别扭，开发也束手束脚。

当了解学习一个新技术新框架新工具的时候，首先要了解其思想。它新在哪里？它提倡了什么理念？它想引导开发者向哪里发展？难道以前的同类技术不行了吗？

这个问题是关于Spring的，也就是关于Java 服务器端技术的，那咱们就举个Java的例子。大家学Java，都知道一本书叫《Thinking In Java》吧？中文叫《Java编程思想》好像是。

Java新手，特别是以前熟悉过程语言的人，首先要理解面向对象的思想，才能用好面向对象的语言。否则你就是用面向过程的语言写着面向过程的程序。你用来用去，就会问自己，和我以前用的那个语言到底有啥区别呢？

不仅仅是编程语言和框架，学习说话语言也是一样。任何人学外语都不会像用自己母语那么自然，关键就是没有用对方语言的思维去组织语言！背再多单词，记再多语法，没有转变到那门语言的思维方式上，就很难说出地道的外语。

生活中也是一样的。这个问题下有用手动挡车和自动挡车来类比Spring Boot和SSM。假如一个人以前开了十几年手动挡汽车，然后换了一部手自一体的，他一开始也不习惯。用自动挡模式驾驶，他的手也会不自觉的去摸档把。最后他把车切换成手动模式才觉得稍微习惯一点。

在日常开发中，经常会见到一种人，他们学习新框架总是要把自己对以前旧框架的认识和经验往上套。套不上就会骂这个东西真难用，套上了就会问到底有啥区别为什么要折腾，这就是底层程序员的通病。

那么Spring Boot和SSM相比，除了之前说过的那些区别，在Web开发上引入了什么新思想？打算把开发者引导到什么方向上去呢？其中重要的一点就是：抛弃服务器端模板，用REST API和前端配合，做到前后端分离的开发。

看我回答过的两个问题：

https://www.zhihu.com/question/306193181/answer/556626638

https://www.zhihu.com/question/309604430/answer/579400528

别人的问题和文章：

https://www.zhihu.com/question/61385975

https://zhuanlan.zhihu.com/p/33155755

总之，Spring Boot开发JSP有一些限制！如果要开发JSP，还要做额外的配置，这不就和Spring Boot减少配置让项目快速启动开发的理念背道而离了吗？不仅仅是限制JSP，而是现在前后端分离的开发就是大势所趋。因为前端越来越专业越来越复杂了，从语言到框架到开发工具已经完全和服务器端不同了，服务器端包含JSP在内的模板语言越来越没有用了。

上面并没有说明确。如果对Spring Framework有最新了解的话，就会知道所谓的“Spring Boot对JSP有一些限制”指的是Spring Boot 结合 Spring MVC的情况下，也就是用spring-boot-starter-web这个Starter的时候。

Spring Boot开发Web应用难道可以不用Spring MVC，还能用其他框架？当然啊，我在回答的第一部分，讲Spring Boot和SSM本质区别的时候就提到了。我们还可以让Spring Boot用spring-boot-starter-webflux这个Starter，用上Spring 5最新提供的Spring WebFlux！关于WebFlux，可以看我另外一个回答：

https://www.zhihu.com/question/294282002/answer/521229241

我们再次看一下Spring官网(http://spring.io)的一副大图(我上面这个回答也用了这幅图)：

![img](https://pic2.zhimg.com/80/v2-2f78f2d7404a11ca5c49da59e98d78d6_720w.jpg?source=1940ef5c)


右侧是传统的Spring MVC，是基于Servlet API的，也就是说至少还支持JSP，只是有一些限制。而左侧是Spring 5最新推出的Spring WebFlux，完全不支持JSP！

看一下Spring Boot文档里分别是怎么说的：

Spring Boot结合Spring MVC部分：

![img](https://pic2.zhimg.com/80/v2-c0f84a043fc4c9887b5e6bded400aea6_720w.jpg?source=1940ef5c)

正式支持的列表里没有JSP，至少提了一下JSP有限制。

> If possible, JSPs should be avoided.
> 如果可能的话，最好别用JSP。

Spring Boot结合Spring WebFlux部分：

![img](https://pic4.zhimg.com/80/v2-b52be9afa190e519b107eca108116b1e_720w.jpg?source=1940ef5c)

根本不支持JSP
压根没提到JSP。

### 再多谈一些

我们再看看Spring官网(http://Spring.io)首页最大最醒目的一幅图。

#### Your App

你的App？难道不是用Spring Boot开发的吗？对。指的是Desktop App、Web App、Mobile App，当然Mobile App也可能是Hybrid或H5开发的。灰色的Your App不是Spring Boot开发的，右侧绿色那三个块块才是。

总之他们都是独立运行在客户端的，他们不是用Spring Boot开发出来的！他们都会向服务器端Spring Boot发送API请求。其中Web App就是我们现在用前端框架写的SPA(Single Page Application)，不是JSP，也不是其他任何服务器端模板语言(FreeMarker、Thymeleaf)！ JSP是服务器端生成HTML，不是客户端技术，而是Java Server Page，是Page，写不出APP。

#### 第一个方块

Spring Boot，Build Anything。为啥是Build Anything？按照绝大多数人的理解，Build Web不就完了吗？就好像我现在正在回答的这个问题大多数其他答主片面肤浅理解的那样，Spring Boot和SSM好像没啥区别啊，都是开发Java Web的，只是配置更方便。

Anything，在我回答的第一部分就已经解释过了。Spring Boot没有必须用Spring MVC，可以不用，也可以换成Spring WebFlux。也可以结合其他专用框架开发应用，都不一定要用Web。

#### 第二个方块

Spring Cloud，就是基于Spring Boot的。

我们看Spring官网(http://spring.io)首页的另一副大图：

![img](https://pic1.zhimg.com/80/v2-67bb8cd121b8414b126ef1f8e6a86361_720w.jpg?source=1940ef5c)


注意最左侧灰色的IoT、Mobile、Browser，同样是之前我们引用的三个方块图里最左侧的“Your App”，右侧绿色的才是Spring Cloud架构里的组成部分。IoT是Internet of Things，物联网。

Spring Cloud架构里面包含的的Service Registry、Service Configuration Center、API Gateway、每一个MicroService，都是基于Spring Boot的应用。

特别是每一个MicroService，不会再用模板引擎，当然也不会用JSP。因为所有前端的API请求都发给API Gateway，Gateway再转发给某个微服务。

我们看看Spring Cloud文档里怎么说的：

> Many of those features are covered by Spring Boot, on which Spring Cloud builds.
> Spring Boot has an opinionated view of how to build an application with Spring. For instance, it has conventional locations for common configuration files and has endpoints for common management and monitoring tasks. Spring Cloud builds on top of that and adds a few features that probably all components in a system would use or occasionally need.

比如我们写一个Spring Cloud Config Server：

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class ConfigServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServiceApplication.class, args);
    }
}
```

比如我们写一个Spring Cloud Eureka Service Registry:

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}
```



比如我们写一个Spring Cloud Zuul Proxy(Gateway):

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {

  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }

}
```

比如我们写一个微服务：

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;

@RestController
@SpringBootApplication
public class SayHelloApplication {

  private static Logger log = LoggerFactory.getLogger(SayHelloApplication.class);

  @RequestMapping(value = "/greeting")
  public String greet() {
    log.info("Access /greeting");

    List<String> greetings = Arrays.asList("Hi there", "Greetings", "Salutations");
    Random rand = new Random();

    int randomNum = rand.nextInt(greetings.size());
    return greetings.get(randomNum);
  }

  @RequestMapping(value = "/")
  public String home() {
    log.info("Access /");
    return "Hi!";
  }

  public static void main(String[] args) {
    SpringApplication.run(SayHelloApplication.class, args);
  }
}
```



这些都要用到@SpringBootApplication

如果MicroService也包含JSP，就产生了这个问题：

https://www.zhihu.com/question/309604430/answer/579400528

这个问题的提出，就是我上文说过的：在日常开发中，经常会见到一种人，他们学习新框架总是要把自己对以前旧框架的认识和经验往上套。套不上就会骂这个东西真难用，套上了就会问到底有啥区别为什么要折腾，这就是底层程序员的通病。

#### 第三个方块

Spring Cloud DataFlow。我们可以写Stream和Task，做数据的抽取、分析、转换等等，他们同样是Spring Boot的应用！这也是Anything包含的，而SSM没这些！

比如我们写一个Task:

```java
@EnableTask
@SpringBootApplication
public class MyTask {

    public static void main(String[] args) {
		SpringApplication.run(MyTask.class, args);
	}
}
```

同样是一个@SpringBootApplication





#### 谈一些更深入的问题



#### 思想要跟上变化

**软件开发，首先是思想活动，其次才是敲代码**。这句话不知道在我的知乎回答里出现了多少遍了，上面的回答内容中也有所体现。

做软件开发不可避免的遇到新语言、新技术、新框架、新工具的出现。当遇到新东西的时候，必须要问自己：**为什么会出现这个东西，以前我开发的好好地，为啥冒出这么个东西来。**

我以前用Struts，现在出来个Spring MVC，Spring这帮人是傻吗？
我现在用Java，Java太强大了，怎么又出来个Kotlin，JetBrains这帮人是活腻了吗？他们想取代Java？
我一直在用JSP，怎么最近几年前端那么火爆？不就是写个页面吗？jQuery不是很好用？那些React、VUE出来干嘛？
我以前用SSM，现在出来个Spring Boot，怎么又搞一个新"框架"啊？这个“框架”感觉就是比SSM配置起来方便点，怎么就这么火呢，做Java服务器端的无人不吹Spring Boot？

整个IT产业到目前还属于朝阳产业。软件开发和编程到现在还处于起步阶段，尤其是国内的程序员都是软件开发行业的第一代人，几乎没有自己的父母就是程序员的。软件开发技术就像婴儿，每天一个变化，千万不要用一成不变的眼光看待它。

所以，新产业就会有新需求，就会有新思想不断涌现去满足那些新需求。当原有技术无法支撑新的思想，就必定涌现出新的东西。

如果有人觉得Struts1还能满足需要，只是需要稍加优化，那就出现Struts2。有另外一拨人觉得Struts根本不能支撑自己的新想法，那就出现Spring MVC了，现在又出现Spring WebFlux了。

如果有人觉得jQuery能够满足自己的需要，那就继续维护增加新功能进去。有人觉得满足不了需求，那就Angularjs、Backbone、Ember出来了，后来又有人觉得这些框架无法支撑自己的想法了，搞出第二代前端框架Angular、React、Vue。

**所以，学习新技术，千万不要用自己以前建立起来的老经验往上生拉硬套！要分清楚哪部分是传承，哪部分是创新。套不上就骂这个框架难用，套上了就问和以前有什么区别。**

以前用SSM，只知道框架。有了Spring Boot就认为它必然就是个新框架而已。然后得出结论：和SSM一样，就是配置简单点。SSM能写的它都能写。殊不知，Spring Boot可能是Java服务器端从来没有出现过的概念，它代表了新理念，不能用任何已经固化的概念去套。

宝宝长出牙，父母就无所适从了，这是又长出一层嘴唇？还是舌头变硬了？反正必须用以前见过的东西网上套。有这样的父母吗？如果没有，那就别用这样的方式去看待软件开发。

#### 为什么不去读文档

这个问题的回答，我引用了多少Spring的文档？而且http://Spring.io首页的三幅图就说明了很多问题。可惜的是，我见到的大多数开发人员没有阅读过官网文档。从来没有访问过http://spring.io首页的都大有人在，更别提去认真读一遍Spring Boot/Sprint Framework/Spring Cloud/Spring Data这些具体项目的文档。



曾经在知乎里回答过一个问题：

https://www.zhihu.com/question/21346206/answer/349792663
我和其他所有该问题下的回答都不同，我不支持初学者一头扎入源代码，具体观点可以去这个回答里看。

就拿我们当前的这个问题来说。你要看多少Spring的源代码才能了解我在回答里里阐述的这些事实，才能对Spring Boot从宏观上有一个全面、清晰的理解？

知乎里有很多类似这样的问题：程序员如何避免走弯路？

**我觉得对于国内很多程序员来说，最大的弯路、冤枉路就是迷信源代码，跳过熟读文档这一步而一头扎进源代码。**

我个人认为的任何成熟软件技术的最佳入门路径都是：

- 先看官方文档。好的技术和框架，官方文档一定全面丰富详实，http://Spring.io就是好文档的典范。所以先把官方文档过一遍，理解的就理解，不理解的要记住在文档的哪一节。
- 开始实践。有些知识只有实践的过程中才能理解，并且加深认识。遇到问题，知道这个问题对应文档的哪一部分，然后去查文档。
- 做完一两个实际项目之后，返回去再读一遍文档，这时你会发现自己站在一个新高度上。
- 1/2/3部分循环...
    越成熟越优秀的开源框架，文档就越丰富越详实。Java这边的Spring 就是最典型的例子，前端的Angular、React都是例子。

如果一个框架文档不齐全，不丰富，说明这个框架本身也不成熟，这个框架的开发团队主要精力还放在功能上，而没有更多地投入到文档上。

#### 别丢掉英语

英语是工具，是你程序员职业生涯上的一对翅膀。大部分一手资料都是英文的。如何慢慢习惯英文阅读，看我曾经回答过一个问题：

软件开发圈的人，一开始是怎么学会使用纯英文描述的 API和各种软件（比如Tomcat）的使用方法的？

https://www.zhihu.com/question/288329542/answer/526126787


看到原来最高赞答案推荐了《Java EE开发的颠覆者：Spring Boot 实战》，我去京东上搜了一下，发现两个问题。

发现Spring Boot最早的中文(原创或翻译)书都是从16年开始陆续上市的。而我14年就开始用Spring Boot，15年初开始用JHipster 2.0了。

我没有看过那些书，所以对书的内容不做评价。但是发现另一个问题，就是这些书都叫Spring Boot实战、Spring Boot揭秘、深入浅出Spring Boot，中文编程的书好像就想不起什么好名字了。尤其是这个”揭秘“。我只在小时候看UFO、玛雅文明、亚特兰蒂斯、克格勃这些书才能看到这个词。Spring Boot有什么秘密可言吗？代码都是开源的！文档随便读！

Spring团队生怕大家不会用，生怕大家不理解，编写了完备的文档，给出了丰富的例子。结果两年后中文书好像还能挖出Spring官方自己都不知道的什么秘密，并且能写厚厚一本。

官方团队在这里给出了八十多个例子：

https://link.zhihu.com/?target=https%3A//github.com/spring-projects/spring-boot/tree/master/spring-boot-samples
长图预警八十多个例子：

![img](https://pic4.zhimg.com/80/v2-dc13f7d95c33457fafc891a9199bb976_720w.jpg?source=1940ef5c)


这些例子涵盖了Spring Boot的方方面面，各种用法各种特性应有尽有。网上第三方的例子还有很多很多。

可悲的是，国内Java服务器端开发人员对如此丰富免费的文档和例子视而不见，却要多等两年，还要花几十几百块钱去买xxx揭秘xxx实战。

更多内容推荐
最后推荐我专栏里的两篇文章：

https://zhuanlan.zhihu.com/p/55173112
https://zhuanlan.zhihu.com/p/55249159
这个系列文章的后续章节会持续更新。

哦，对了，学习Spring Boot并且想充分把握和发扬Spring Boot的优势，可以学一下JHipster。JHipster本身以及它包含的所有最佳实践(Best Practice)，都是我们使用Spring Boot时候应该学习和应用的。

如果说Spring Boot能够让你的快速搭建Java服务器端应用，那么JHipster则把这种理念发展到了全栈。服务器端继续使用Spring Boot，前端使用Angular/React/VUE，让你快速搭建一个全栈应用！

