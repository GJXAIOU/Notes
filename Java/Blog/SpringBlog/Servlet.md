# Servlet（上）


## Servlet的前世今生

类似于 Servlet 是 **Server Applet**（运行在服务端的小程序）等其他博文已经提过的内容，这里就不重复了。它就是用来处理浏览器请求的。

之前在 [Tomcat外传](https://zhuanlan.zhihu.com/p/54121733)中我们聊过，所谓 Tomcat 其实是 Web 服务器和 Servlet 容器的结合体。

什么是 Web 服务器？

比如，我当前在杭州，你能否用自己的电脑访问我桌面上的一张图片？恐怕不行。我们太习惯通过URL访问一个网站、下载一部电影了。一个资源，如果没有 URL 映射，那么外界几乎很难访问。而 Web 服务器的作用说穿了就是：将某个主机上的资源映射为一个 URL 供外界访问。

什么是Servlet容器？

Servlet 容器，顾名思义里面存放着 Servlet 对象。我们为什么能通过Web服务器映射的URL访问资源？肯定需要写程序处理请求，主要3个过程：

- 接收请求
- 处理请求
- 响应请求

任何一个应用程序，必然包括这三个步骤。其中接收请求和响应请求是共性功能，且没有差异性。访问淘宝和访问京东，都是接收 [www.taobao.com/brandNo=1](https://link.zhihu.com/?target=http%3A//www.taobao.com/brandNo%3D1)，响应给浏览器的都是 JSON 数据。于是，大家就把接收和响应两个步骤抽取成 Web 服务器：

![img](https://pic4.zhimg.com/80/v2-3d86f470ec1dc31bbe93d1df2c30fa47_720w.jpg)

但处理请求的逻辑是不同的。没关系，抽取出来做成 Servlet，交给程序员自己编写。当然，随着后期互联网发展，出现了三层架构，所以一些逻辑就从 Servlet 抽取出来，分担到 Service 和 Dao。

![img](https://pic2.zhimg.com/80/v2-f41587429ebde63225029b0c235960c1_720w.jpg)Java Web 开发经典三层架构：代码分层，逻辑清晰，还解耦

但是 Servlet 并不擅长往浏览器输出 HTML 页面，所以出现了 JSP。JSP 的故事，我已经在[浅谈 JSP](https://zhuanlan.zhihu.com/p/42343690)讲过了。

等 Spring 家族出现后，Servlet 开始退居幕后，取而代之的是方便的 SpringMVC。SpringMVC 的核心组件DispatcherServlet 其实本质就是一个 Servlet。但它已经自立门户，在原来 HttpServlet 的基础上，又封装了一条逻辑。

总之，很多新手程序员框架用久了，甚至觉得 Spring MVC 就是 Spring MVC，和 Servlet 没半毛钱关系。没关系，看完以后你就想起来被 Servlet 支配的恐惧了。

------

## 我所理解的 Java Web 三大组件

不知道从什么时候开始，我们已经不再关心、甚至根本不知道到底谁调用了我写的这个程序，反正我写了一个类，甚至从来没 new 过，它就跑起来了...

我们把模糊的记忆往前推一推，没错，就是在学了 Tomcat 后！从 Tomcat 开始，我们再也没写过 main 方法。以前，一个main 方法启动，程序间的调用井然有序，我们知道程序所有流转过程。

但是到了 Java web 后，Servlet/Filter/Listener 一路下来我们越学越沮丧。没有 main，也没有 new，写一个类然后在web.xml 中配个标签，它们就这么兀自运行了。

其实，这一切的一切，简单来说就是“注入”和“回调”。想象一下 Tomcat 里有个 main 方法，假设是这样的：

![img](https://pic3.zhimg.com/80/v2-ce6e39bb02e3c6a2f4eb1e5afaa6e4e6_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-14c18b69b5fb642f8d56698d2df20171_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-d473a8662d758859e75c3f9afce9e982_720w.jpg)

其实，编程学习越往后越是如此，我们能做的其实很有限。大部分工作，框架都已经帮我们做了。只要我们实现 xxx 接口，它会帮我们创建实例，然后搬运（接口注入）到它合适的位置，然后一套既定的流程下来，肯定会执行到。我只能用中国一个古老的成语形容这种开发模式：闭门造车，出门合辙（敲黑板，成语本身是夸技术牛逼，而不是说某人瞎几把搞）。

很多时候，框架就像一个傀儡师，我们写的程序是傀儡，顶多就是给傀儡化化妆、打扮打扮，实际的运作全是傀儡师搞的。



了解到这个层面后，Java Web 三大组件任何生命周期相关的方法、以及调用时 Tomcat 传入的形参，这里就不再强调。肯定是程序的某处，在创建实例后紧接着就传入参数调用了呗。没啥神秘的。

------

## 如何编写一个 Servlet

首先，我们心里必须有一个信念：我们都是菜鸡，框架肯定不会让我们写很难的代码。

进入 Tomcat 阶段后，我们开始全面面向接口编程。但是“面向接口编程”这个概念，最早其实出现在 JDBC 阶段。我就问你，JDBC 接口是你自己实现的吗？别闹了，你导入 MySQL 的驱动包，它给你搞定了一切。

真正的连接过程太难写了，朋友们。底层就是 TCP 连接数据库啊，你会吗？写 Socket，然后进行数据库校验，最后返回Connection？这显然超出我们的能力范围了。我们这么菜，JDBC 不可能让我们自己动手的。所以各大数据库厂商体贴地推出了驱动包，里面有个 Driver 类，调用

```text
driver.connect(url, username, password);
```

即可得到 Connection。

BUT，这一次难得 Tomcat 竟然这么瞧得起我黄某 ，仅仅提供了 javax.servlet 接口，这是打算让我自己去实现？

不，不可能的，肯定是因为太简单了。

查看接口方法：

![img](https://pic4.zhimg.com/80/v2-cbb35ec0c92292ea599108e643fd5183_720w.jpg)

五个方法，最难的地方在于形参，然而 Tomcat 会事先把形参对象封装好传给我...除此以外，既不需要我写 TCP 连接数据库，也不需要我解析 HTTP 请求，更不需要我把结果转成 HTTP 响应，request 对象和 response 对象帮我搞定了。

看吧，Tomcat 是不是把我们当成智障啊。

Tomcat 之所以放心地交给我们实现，是**因为 Servlet 里主要写的代码都是业务逻辑代码**。和原始的、底层的解析、连接等没有丝毫关系。最难的几个操作，人家已经给你封装成形参传进来了。

也就是说，**Servlet 虽然是个接口，但实现类只是个空壳，我们写点业务逻辑就好了。**

![img](https://pic1.zhimg.com/80/v2-1a911529c489ebdcb2a17a8e19d87290_720w.jpg)

总的来说，Tomcat 已经替我们完成了所有“菜鸡程序员搞不定的骚操作”，并且传入三个对象：ServletConfig、ServletRequest、ServletResponse。接下来，我们看看这三个传进来都是啥。



**ServletConfig**

翻译过来就是“Servlet 配置”。我们在哪配置 Servlet 来着？web.xml 嘛。请问你会用 dom4j 解析 xml 得到对象吗？

可能...会吧，就是不熟练，嘿嘿嘿。

所以，Tomcat 还真没错怪我们，已经帮“菜鸡们”搞掂啦：

![img](https://pic1.zhimg.com/80/v2-3dd656100783b3e9e62621ad8e2e9b04_720w.jpg)

也就是说，servletConfig 对象封装了 servlet 的一些参数信息。如果需要，我们可以从它获取。



**Request/Response**

两位老朋友，不用多介绍了。其实，很多人看待 HTTP 和 Request/Response 的眼光过于分裂。它们的关系就像菜园里的大白菜和餐桌上的酸辣白菜一样。**HTTP 请求到了 Tomcat 后，Tomcat 通过字符串解析，把各个请求头（Header），请求地址（URL），请求参数（QueryString）都封装进了 Request 对象中**。通过调用

```text
request.getHeader();
request.getUrl()；
request.getQueryString();
...
```

等等方法，都可以得到浏览器当初发送的请求信息。

至于 Response，Tomcat 传给 Servlet 时，它还是空的对象。Servlet 逻辑处理后得到结果，最终通过 response.write() 方法，将结果写入 response 内部的缓冲区。Tomcat 会在 servlet 处理结束后，拿到 response，遍历里面的信息，组装成HTTP 响应发给客户端。

![img](https://pic1.zhimg.com/80/v2-7405fb1912570c73de8dd76da725b17c_720w.jpg)

**Servlet 接口 5 个方法，其中 init、service、destroy 是生命周期方 法。init 和destroy 各自只执行一次，即 servlet 创建和销毁时。而 service 会在每次有新请求到来时被调用。也就是说，我们主要的业务代码需要写在 service 中。**

但是，浏览器发送请求最基本的有两种：Get/Post，于是我们必须这样写：

![img](https://pic1.zhimg.com/80/v2-94e6aac29c7bb1353020d2df7f422d58_720w.jpg)

很烦啊。有没有办法简化这个操作啊？我不想直接实现 javax.servlet 接口啊。



于是，菜鸡程序员找了下，发现了GenericServlet，是个**抽象类**。

![img](https://pic2.zhimg.com/80/v2-b9f65e77009de2832d721cb28d5ae6f1_720w.jpg)

我们发现GenericServlet做了以下改良：

- 提升了init方法中原本是形参的servletConfig对象的作用域（成员变量），方便其他方法使用
- init方法中还调用了一个init空参方法，如果我们希望在servlet创建时做一些什么初始化操作，可以继承GenericServlet后，覆盖init空参方法
- 由于其他方法内也可以使用servletConfig，于是写了一个getServletContext方法
- service竟然没实现...要它何用

放弃GenericServlet。

于是我们继续寻找，又发现了HttpServlet：

![img](https://pic1.zhimg.com/80/v2-869e7ef9c2c7aaf4b90572ca140bd1b4_720w.jpg)

它继承了GenericServlet。

GenericServlet本身是一个抽象类，有一个抽象方法service。查看源码发现，HttpServlet已经实现了service方法：

![img](https://pic1.zhimg.com/80/v2-73b703e690ce018ffe88280376a67dc0_720w.jpg)

好了，也就是说HttpServlet的service方法已经替我们完成了复杂的请求方法判断。

但是，我翻遍整个HttpServlet源码，都没有找出一个抽象方法。所以为什么HttpServlet还要声明成抽象类呢？

看一下HttpServlet的文档注释：

![img](https://pic1.zhimg.com/80/v2-b6602e7e7c24862f54041e3998bd7ee0_720w.jpg)

一个类声明成抽象方法，一般有两个原因：

- 有抽象方法
- 没有抽象方法，但是不希望被实例化

HttpServlet做成抽象类，仅仅是为了不让new。

![img](https://pic3.zhimg.com/80/v2-1c6bf76d5f9510e8dcd58fb6b870c95a_720w.jpg)所以构造方法不做任何事

它为什么不希望被实例化，且要求子类重写doGet、doPost等方法呢？

我们来看一下源码：

![img](https://pic4.zhimg.com/80/v2-ab9504d35eab343a522926bf18c3167b_720w.jpg)protected修饰，希望子类重写

如果我们没重写会怎样？

![img](https://pic2.zhimg.com/80/v2-fb86edf9aaac3049028e25fa022a9811_720w.jpg)

浏览器页面会显示：405（http.method_get_not_supported）

也就是说，HttpServlet虽然在service中帮我们写了请求方式的判断。但是针对每一种请求，业务逻辑代码是不同的，HttpServlet无法知晓子类想干嘛，所以就抽出七个方法，并且提供了默认实现：报405、400错误，提示请求不支持。

但这种实现本身非常鸡肋，简单来说就是等于没有。所以，不能让它被实例化，不然调用doXxx方法是无用功。

Filter用到了责任链模式，Listener用到了观察者模式，Servlet也不会放过使用设计模式的机会：模板方法模式。上面的就是。这个模式我在JDBC的博文里讲过了：[JDBC（中）](https://zhuanlan.zhihu.com/p/64749458)

![img](https://pic2.zhimg.com/80/v2-b33c2c238803958be0bfa70d0f40c211_720w.jpg)

小结：

- 如何写一个Servet？
- 不用实现javax.servlet接口
- 不用继承GenericServlet抽象类
- 只需继承HttpServlet并重写doGet()/doPost()
- 父类把能写的逻辑都写完，把不确定的业务代码抽成一个方法，调用它。当子类重写该方法，整个业务代码就活了。这就是模板方法模式

![img](https://pic2.zhimg.com/80/v2-250e370a7548fa65ed70d73ff82f2829_720w.jpg)

下篇聊聊ServletContext和Servlet映射路径。

2019-5-13 19:35:38



博主耗费一年时间编写的Java进阶小册已经上线，覆盖日常开发所需大部分技术，且通俗易懂、深入浅出、图文丰富，需要的同学请戳：

[中级Java程序员如何进阶（小册）698 赞同 · 99 评论文章![img](https://pic1.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/212191791)

2021-07-06

编辑于 2021-07-06 08:30



# Servlet（下）

[![bravo1988](https://pic2.zhimg.com/v2-1907eb21be63d35b077e6ed3cbcbfe13_xs.jpg?source=172ae18b)](https://www.zhihu.com/people/huangsunting)

[bravo1988](https://www.zhihu.com/people/huangsunting)

Java进阶小册已上线，详见动态置顶，助力野生程序员

关注他

564 人赞同了该文章

在这一篇文章里，将会讨论ServletContext以及Servlet映射规则。这两个知识点非常重要，ServletContext直接关系到SpringIOC容器的初始化（请参考[ContextLoaderListener解析](https://zhuanlan.zhihu.com/p/65258266)），而Servlet映射规则与SpringMVC关系密切。

可以说，作为初学者只要把这两点搞清楚，那么对Spring/SpringMVC的理解将会超过70%的程序员。我没开玩笑，你随便抓一个身边的同事问问，Tomcat和Spring什么关系？SpringMVC和Servlet什么关系？估计没几个能给你讲清楚的。

我会先把SpringMVC讲得很简单，等大家觉得它就是个Servlet的时候，我又会把SpringMVC慢慢展开，露出它的全貌。此时你又会发现：SpringMVC is not only a Servlet.

主要内容：

- ServletContext是什么
- 如何获取ServletContext
- Filter拦截方式之：REQUEST/FORWARD/INCLUDE/ERROR
- Servlet映射器
- 自定义DispatcherServlet
- DispatcherServlet与SpringMVC
- conf/web.xml与应用的web.xml

------

## ServletContext是什么

ServletContext，直译的话叫做“Servlet上下文”，听着挺别扭。它其实就是个大容器，是个map。服务器会为每个应用创建一个ServletContext对象：

- ServletContext对象的创建是在服务器启动时完成的
- ServletContext对象的销毁是在服务器关闭时完成的

![img](https://pic2.zhimg.com/80/v2-40ed984999cab23bc4e9e17a39e84839_720w.jpg)

ServletContext对象的作用是在整个Web应用的动态资源（Servlet/JSP）之间共享数据。例如在AServlet中向ServletContext对象保存一个值，然后在BServlet中就可以获取这个值。

![img](https://pic1.zhimg.com/80/v2-291c4b8583663764b091fa2acd37e724_720w.jpg)

这种用来装载共享数据的对象，在JavaWeb中共有4个，而且更习惯被成为“域对象”：

- ServletContext域（Servlet间共享数据）
- Session域（一次会话间共享数据，也可以理解为多次请求间共享数据）
- Request域（同一次请求共享数据）
- Page域（JSP页面内共享数据）

它们都可以看做是map，都有getAttribute()/setAttribute()方法。

来看一下物理磁盘中的配置文件与内存对象之间的映射关系

![img](https://pic4.zhimg.com/80/v2-2530b17c1ee7e94bbcce7ca472d6a667_720w.jpg)每一个动态web工程，都应该在WEB-INF下创建一个web.xml，它代表当前整个应用。Tomcat会根据这个配置文件创建ServletContext对象

------

## 如何获取ServletContext

还记得GenericServlet吗？它在init方法中，将Tomcat传入的ServletConfig对象的作用域由局部变量（方法内使用）提升到成员变量。并且新建了一个getServletContext()：

![img](https://pic4.zhimg.com/80/v2-e4fc1f0a49a67b59256b4935a47ac913_720w.jpg)getServletContext()内部其实就是config.getServletContext()

也就是说ServletConfig对象可以得到ServletContext对象。但是这并不意味这ServletConfig对象包含着ServletContext对象，而是ServletConfig维系着ServletContext的引用。

其实这也很好理解：servletConfig是servletContext的一部分，就像他儿子。你问它父亲是谁，它当然能告诉你。

另外，Session域和Request域也可以得到ServletContext

```text
session.getServletContext();
request.getServletContext();
```

![img](https://pic4.zhimg.com/80/v2-5b2c02d7dac0cd170c0194679f4d9483_720w.jpg)用域对象获得域对象

最后，还有个冷门的，我在监听器那一篇讲过了：

![img](https://pic2.zhimg.com/80/v2-ec7e00121cdd1df01898f9b91d27c60d_720w.jpg)事件对象就是对事件源（被监听对象）的简单包装

所以，获取ServletContext的方法共5种（page域这里不考虑，JSP太少用了）：

- ServletConfig#getServletContext();
- GenericServlet#getServletContext();
- HttpSession#getServletContext();
- HttpServletRequest#getServletContext();
- ServletContextEvent#getServletContext();

------

## Filter拦截方式之：REQUEST/FORWARD/INCLUDE/ERROR

在很多人眼里，Filter只能拦截Request：

![img](https://pic2.zhimg.com/80/v2-b8dfca0f5a4895bce75c2ce6b6f0c725_720w.jpg)

这样的理解还是太片面了。

其实配置Filter时可以设置4种拦截方式：

![img](https://pic2.zhimg.com/80/v2-bbd582e840cac5e82eda33bd91cf60bd_720w.jpg)

很多人要么不知道，要么理解得不够清晰。

这里，我先帮大家把这4种方式和重定向（Redirect）剥离开，免得有人搞混，它们是完全两类（前者和Request有关，后者通过Response发起），以FORWARD为例：

![img](https://pic2.zhimg.com/80/v2-70f251139437bb33e2f7a93dfa89c135_720w.jpg)蓝色：重定向，橙色：转发

我们日常开发中，FORWARD用的最多的场景就是转发给JSP，然后模板输出HTML。

Redirect和REQUEST/FORWARD/INCLUDE/ERROR最大区别在于：

- 重定向会导致浏览器发送**2**次请求，FORWARD们是服务器内部的**1**次请求



了解这个区别之后，我提一个很奇怪的问题：为什么这4种只引发1次请求？

是不是听傻了？我接下来给的答案，属于意料之外情理之中的那种：

> 因为FORWARD/INCLUDE等请求的分发是服务器内部的流程，不涉及浏览器

还记得如何转发吗：

![img](https://pic2.zhimg.com/80/v2-425f3e0e15abf4743546fff21c5d9999_720w.jpg)

我们发现通过Request或者ServletContext都可以得到分发器Dispatcher，但由于ServletContext代表整个应用，我更倾向于认为：ServletContext拥有分发器，Request是找它借的。

分发器是干嘛的？分发请求：REQUEST/FORWARD/INCLUDE/ERROR。REQUEST是浏览器发起的，而ERROR是发生页面错误时发生的，稍微特殊些。

所以，所谓Filter更详细的拦截其实是这样：

![img](https://pic3.zhimg.com/80/v2-1d6b0e77752d60f39d829ad39a4a630a_720w.jpg)灰色块：Filter

最外层那个圈，可以理解成ServletContext，FORWARD/INCLUDE这些都是内部请求。如果在web.xml中配置Filter时4种拦截方式全配上，那么服务器内部的分发跳转都会被过滤。

当然，这些都是可配置的，默认只拦截REQUEST，也就是浏览器来的那一次。

------

## **Servlet映射器**

上一篇说了很多Servlet的源码，也介绍了Servlet的作用就是处理请求。但是对于每个请求具体由哪个Servlet处理，却只字未提。其实，每一个URL要交给哪个Servlet处理，具体的映射规则都由一个映射器决定：

![img](https://pic4.zhimg.com/80/v2-ff68a99ab3c8a9d9b2bffbc51e22608b_720w.jpg)

这所谓的映射器，其实就是Tomcat中一个叫Mapper的类。

![img](https://pic4.zhimg.com/80/v2-97f7eccfeddccaa932d30d9c23a1af1b_720w.jpg)

它里面有个internalMapWrapper方法：

![img](https://pic2.zhimg.com/80/v2-6de02a1543ac243e79adc217012418ad_720w.jpg)

定义了7种映射规则：

![img](https://pic3.zhimg.com/80/v2-0b486de6085fa9eae71b9203c5560236_720w.jpg)1.精确匹配 2.前缀匹配

![img](https://pic2.zhimg.com/80/v2-27bf362804b224137c0e51d5d17c8639_720w.jpg)3.扩展名匹配

![img](https://pic3.zhimg.com/80/v2-e46693fe4049808ba53c4577db61fa12_720w.jpg)4.5.6 欢迎列表资源匹配？

![img](https://pic1.zhimg.com/80/v2-c7ff7422c3a6ac49559e6494b7351590_720w.png)7.如果上面都不匹配，则交给DefaultServlet，就是简单地用IO流读取静态资源并响应给浏览器。如果资源找不到，报404错误

简单来说就是：

> 对于静态资源，Tomcat最后会交由一个叫做DefaultServlet的类来处理
> 对于Servlet ，Tomcat最后会交由一个叫做 InvokerServlet的类来处理
> 对于JSP，Tomcat最后会交由一个叫做JspServlet的类来处理
> 引用自：[tomcat中对静态资源的访问也会用servlet来处理吗？](https://www.zhihu.com/question/57400909/answer/154753720)

![img](https://pic1.zhimg.com/80/v2-42c3d43b3b7dd56851d1018d2186d1f0_720w.jpg)

------

## 自定义DispatcherServlet

web.xml

![img](https://pic3.zhimg.com/80/v2-98fb7efadcc537b91fb57b00a100285a_720w.jpg)

DispatcherServlet

![img](https://pic2.zhimg.com/80/v2-4102042f7987731d511b905079908581_720w.jpg)

知道了映射器的映射规则后，我们来分析下上图中三种拦截方式会发生什么。

但在此之前，我必须再次强调，我从没说我现在写的是SpringMVC的DispatcherServlet，**这是我自己自定义的一个普通Servlet，**恰好名字叫DispatcherServlet而已。所以，下面的内容，请当做一个普通Servlet的映射分析。

- ***.do：拦截.do结尾**

![img](https://pic4.zhimg.com/80/v2-042f720df36b6102a380fde37c5b79c3_720w.jpg)

各个Servlet和谐相处，没问题。



- **/\*：拦截所有**

![img](https://pic4.zhimg.com/80/v2-b09b1c9c18463d995ef3fd8172725047_720w.jpg)

拦截localhost:8080

![img](https://pic2.zhimg.com/80/v2-c9d5eed6c2a965d61fd36781c1184d95_720w.jpg)

拦截localhost:8080/index.html

![img](https://pic2.zhimg.com/80/v2-5cb839a2c50ffa580ad3621303ec48d5_720w.jpg)

拦截localhost:8080/index.jsp

![img](https://pic1.zhimg.com/80/v2-701e3e3be828b7948bdb74840944dcb4_720w.jpg)

也就是说，/*这种配置，相当于把DefaultServlet、JspServlet以及我们自己写的其他Servlet都“短路”了，它们都失效了。

这会导致**两个问题**：

- JSP无法被编译成Servlet输出HTML片段（JspServlet短路）
- HTML/CSS/JS/PNG等资源无法获取（DefaultServlet短路）



- **/：拦截所有，但不包括JSP**

![img](https://pic1.zhimg.com/80/v2-e132fe10fc71bd79ce2d1f79964860d4_720w.jpg)

拦截localhost:8080

![img](https://pic2.zhimg.com/80/v2-4df939e9e882ba0fa7c5b74d2fa92329_720w.jpg)

拦截localhost:8080/index.html

![img](https://pic1.zhimg.com/80/v2-d6a36a15180323701c78d0866051dafc_720w.jpg)

不拦截JSP

![img](https://pic1.zhimg.com/80/v2-702efdae9e66ab5de9c71edbf6d0f528_720w.jpg)

虽然JSP不拦截了，但是DefaultServlet还是“短路”了。而DispatcherServlet把本属于DefaultServlet的工作也抢过来，却又不会处理（IO读取静态资源返回）。

怎么办？

------

## DispatcherServlet与SpringMVC

SpringMVC的核心控制器叫DispatcherServlet，映射原理和我们上面山寨版的一样，因为本质还是个Servlet。但SpringMVC提供了一个标签，解决上面/无法读取静态资源的问题：

```text
    <!-- 静态资源处理  css js imgs -->
    <mvc:resources location="/resources/**" mapping="/resources"/>
```

其他的我也不说了，一张图，大家体会一下DispatcherServlet与SpringMVC到底是什么关系：

![img](https://pic4.zhimg.com/80/v2-ff3412893ecd4b0737ecd2a40447d48f_720w.jpg)

DispatcherServlet确实是一个Servlet，但它只是入口，SpringMVC要比想象的庞大。

------

## conf/web.xml与应用的web.xml

conf/web.xml指的是Tomcat全局配置web.xml。

![img](https://pic4.zhimg.com/80/v2-a6be039280a88dd07d31ec5b87c03e13_720w.jpg)

它里面配置了两个Servlet：

![img](https://pic4.zhimg.com/80/v2-4262abc58a64cfb800952ceb23355c23_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-d56fa3e8b050cb0cbc78df926528d2f9_720w.jpg)

也就是JspServlet和DefaultServlet的映射路径。

我们可以按Java中“继承”的思维理解conf/web.xml：

> conf/web.xml中的配置相当于写在了每一个应用的web.xml中。

相当于每个应用默认都配置了JSPServlet和DefaultServlet处理JSP和静态资源。

如果我们在应用的web.xml中为DispatcherServlet配置/，会和DefaultServlet产生路径冲突，从而覆盖DefaultServlet。此时，所有对静态资源的请求，映射器都会分发给我们自己写的DispatcherServlet处理。遗憾的是，它只写了业务代码，并不能IO读取并返回静态资源。JspServlet的映射路径没有被覆盖，所以动态资源照常响应。

如果我们在应用的web.xml中为DispatcherServlet配置/*，虽然JspServlet和DefaultServlet拦截路径还是.jsp和/，没有被覆盖，但无奈的是在到达它们之前，请求已经被DispatcherServlet抢去，所以最终不仅无法处理JSP，也无法处理静态资源。

2019-5-14 18:19:05

博主耗费一年时间编写的Java进阶小册已经上线，覆盖日常开发所需大部分技术，且通俗易懂、深入浅出、图文丰富，需要的同学请戳：

[bravo1988：中级Java程序员如何进阶（小册）698 赞同 · 99 评论文章](https://zhuanlan.zhihu.com/p/212191791)

2021-07-06

编辑于 2021-07-06 08:24
