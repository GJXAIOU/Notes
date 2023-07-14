# Java Web(一) Servlet详解
[原文地址](https://www.cnblogs.com/whgk/p/6399262.html)

@toc
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　

## 一、Servlet 含义：
　**处理请求和发送响应的过程是由一种叫做Servlet的程序来完成的**，并且Servlet是为了解决**实现动态页面**而衍生的东西。理解这个的前提是了解一些http协议的东西，并且知道B/S模式(浏览器/服务器)。

　B/S:浏览器/服务器。 浏览器通过网址来访问服务器，比如访问百度，在浏览器中输入www.baidu.com，这个时候浏览器就会显示百度的首页，那么这个具体的过程，步骤是怎样的呢？这个就了解一下http请求和响应了

![HTTP过程]($resource/HTTP%E8%BF%87%E7%A8%8B.png)

请求，响应：通过给的链接应该可以知道这两个具体的内容

## 二、 Tomcat和 Servlet的关系

　**Tomcat 是Web应用服务器,是一个Servlet/JSP容器**. Tomcat 作为Servlet容器,**负责处理客户请求,把请求传送给Servlet,并将Servlet的响应传送回给客户**.而**Servlet是一种运行在支持Java语言的服务器上的组件**. Servlet最常见的用途是扩展Java Web服务器功能,提供非常安全的,可移植的,易于使用的CGI替代品.

　从http协议中的请求和响应可以得知，浏览器发出的请求是一个请求文本，而浏览器接收到的也应该是一个响应文本。但是在上面这个图中，并不知道是如何转变的，只知道浏览器发送过来的请求也就是request，我们响应回去的就用response。忽略了其中的细节，现在就来探究一下。
　　　　　　　　　　　　
![Tomcat服务器响应客户请求过程]($resource/Tomcat%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%93%8D%E5%BA%94%E5%AE%A2%E6%88%B7%E8%AF%B7%E6%B1%82%E8%BF%87%E7%A8%8B.png)
　　
　　==☆☆☆☆☆==①：Tomcat将http请求文本接收并解析，然后封装成HttpServletRequest类型的request对象，所有的HTTP头数据读可以通过request对象调用对应的方法查询到。
　　==☆☆☆☆☆==②：Tomcat同时会将要响应的信息封装为HttpServletResponse类型的response对象，通过设置response属性就可以控制要输出到浏览器的内容，然后将response交给 Tomcat， Tomcat就会将其变成响应文本的格式发送给浏览器

　　Java Servlet API 是Servlet容器( Tomcat)和Servlet之间的接口，它定义了serlvet的各种方法，还定义了Servlet容器传送给Servlet的对象类，其中最重要的就是ServletRequest和ServletResponse。所以说我们在编写Servlet时，需要实现Servlet接口，按照其规范进行操作。

## 三、编写Servlet

### (一)手动编写Servlet。

1、创建一个MyServlet继承HttpServlet，重写doGet和doPost方法，也就是看请求的方式是get还是post，然后用不同的处理方式来处理请求，

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 做get请求的处理
        System.out.println("get");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 做POST请求的处理
        doGet(req, resp);
    }
}
```


2、在web.xml中配置MyServlet，为什么需要配置？让浏览器发出的请求知道到达哪个Servlet，也就是让 Tomcat将封装好的request找到对应的Servlet让其使用。
配置四个东西。
```xml
<servlet>
     <!-- servlet的内部名称 这个名字可以自定义,一般和Servlet类名相同-->
     <servlet-name>MyServlet</servlet-name>
     <!-- Servlet全限定类名，也就是包名.类名 也是Servlet位置-->
     <servlet-class>demo.MyServlet</servlet-class>
 </servlet>
<servlet-mapping>
     <!-- servlet的名字 【要求和上边的名字必须一模一样】-->
     <servlet-name>HelloServlet</servlet-name>
     <!-- servlet的访问名称: http://localhost:8080/mywebs/访问名 ,浏览器通过这个URL找到Servlet-->
     <url-pattern>/Myservlet</url-pattern>
 </servlet-mapping>
```

配置之后，浏览器是如何通过我们配置的信息来找到对应的Servlet的。
![浏览器通过配置信息找到Servlet]($resource/%E6%B5%8F%E8%A7%88%E5%99%A8%E9%80%9A%E8%BF%87%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%E6%89%BE%E5%88%B0Servlet.png)

解释： 按照步骤，首先浏览器通过http://localhost:8080/test01/MyServlet来找到web.xml中的url-pattern，这就是第一步，匹配到了url-pattern后，就会找到第二步Servlet的名字MyServlet，知道了名字，就可以通过Servlet-name找到第三步，到了第三步，也就能够知道Servlet的位置了。然后到其中找到对应的处理方式进行处理。

3、实验，验证上面配置成功。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216094649550-2096214829.png)　　　

　![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216094700035-984507451.png)

### （二) 利用向导新建MyServlet

这个就相对简单了，web.xml不用我们手动配置，工具直接帮我们自动配置了

1、右击项目，在new选项中有直接新建Servlet的选项

2、配置MyServlet类中的信息

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216095540332-191821755.png)　

3、配置web.xml中的Servlet信息

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216095633754-1304935718.png)　　　

4、查看MyServle01类中的代码和web.xml，其中的配置跟手动的配置是一样的，只是用图形化界面，让我们更方便的创建Servlet而产生的。

## 三、详解创建Servlet的原理

### （一）Servlet的生命周期是什么？

服务器启动时(web.xml中配置load-on-startup=1，默认为0)或者第一次请求该Servlet时，就会初始化一个Servlet对象，也就是会执行初始化方法init(ServletConfig conf)，这个生命周期只执行一遍；

该Servlet对象去处理所有客户端请求，在service(ServletRequest req，ServletResponse res)方法中执行；
最后服务器关闭时，才会销毁这个Servlet对象，执行destroy()方法。整个生命周期只执行一遍；

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216103737254-1072057229.png)

### （二）为什么创建的Servlet是继承自httpServlet，而不是直接实现Servlet接口？

### （三）Servlet的生命周期中，可以看出，执行的是service方法，为什么我们就只需要写doGet和doPost方法呢？

查看源码，httpServlet的继承结构。

httpServlet继承GenericServlet。懂的人立马就应该知道，GenericServlet(通用Servlet)的作用是什么？大概的就是将实现Servlet接口的方法，简化编写Servlet的步骤。具体下面详解
`public abstract class HttpServlet extends GenericServlet`

GenericServlet的继承结构，实现了Servlet接口和ServletConfig接口，
`public abstract class GenericServlet implements Servlet, ServletConfig, Serializable`
Servlet接口内容

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216142610660-322166979.png)　

从这里可以看到，Servlet生命周期的三个关键方法，init、service、destroy。还有另外两个方法，一个getServletConfig()方法来获取ServletConfig对象，ServletConfig对象可以获取到Servlet的一些信息，ServletName、ServletContext、InitParameter、InitParameterNames、通过查看ServletConfig这个接口就可以知道

ServletConfig接口内容
![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216143011050-806963452.png)

其中ServletContext对象是Servlet上下文对象，功能有很多，获得了ServletContext对象，就能获取大部分我们需要的信息，比如获取Servlet的路径，等方法。

到此，就知道了Servlet接口中的内容和作用，总结起来就是，三个生命周期运行的方法，获取ServletConfig，而通过ServletConfig又可以获取到ServletContext。而GenericServlet实现了Servlet接口后，也就说明我们可以直接继承GenericServlet，就可以使用上面我们所介绍Servlet接口中的那几个方法了，能拿到ServletConfig，也可以拿到ServletContext，不过那样太麻烦，不能直接获取ServletContext，所以GenericServlet除了实现Servlet接口外，还实现了ServletConfig接口，那样，就可以直接获取ServletContext了。

GenericServlet类的内容详解

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216145502191-411577839.png)

看上图，用红色框框起来的就是实现Servlet和ServletConfig接口所实现的方法，有9个，这很正常，但是我们可以发现，init方法有两个，一个是带有参数ServletConfig的，一个有无参的方法，为什么这样设计？这里需要知道其中做了什么事情，来看看这两个方法分别做了什么事？

init(ServletConfig config)
```java
public void init(ServletConfig config) throws ServletException {
      this.config = config;
      this.init();
  } 
```

init()
```java
public void init() throws ServletException {
  }
```


一个成员变量config
`private transient ServletConfig config;`

getServletConfig()

`public ServletConfig getServletConfig() {return this.config; }`

通过这几个方法一起来讲解，首先看init(ServletConfig config)方法，因为只有init(ServletConfig config)中带有ServletConfig对象，为了方便能够在其他地方也能直接使用ServletConfig对象，而不仅仅局限在init(ServletConfig config)方法中，所以创建一个私有的成员变量config，在init(ServletConfig config)方法中就将其赋值给config，然后通过getServletConfig()方法就能够获取ServletConfig对象了，这个可以理解，但是在init(ServletConfig config)中，158行，还调用了一个init()方法，并且这个init()方法是空的，什么读没有，这是为什么呢？这个原因是为了防止一件事情，当我们需要在init方法中做一点别的事情，我们想到的方法就是继承GenericServlet并且重写了init(ServletConfig config)方法，这样依赖，就破坏了原本在GenericServlet类中init(ServletConfig config)写的代码了，也就是在GenericServlet类中的成员变量config会一直是null，无法得到赋值，因为被重写了，就不会在执行GenericServlet中init(ServletConfig config)方法中的代码。要想赋值，就必须在重写的init(ServletConfig config)方法中调用父类的init(ServletConfig config)方法，也就是super.init(ServletConfig config)，这样一来，就很不方便，怕有时候会忘了写这句代码，所以在GenericServlet类中增加一个init()方法，以后需要在init方法中需要初始化别的数据，只需要重写init()这个方法，而不需要去覆盖init(ServletConfig config)这个方法，这样设计，就好很多，不用在管init(ServletConfig config)这个其中的内容了。也不用出现其他的问题。

service(ServletRequest req, ServletResponse res)
```java
public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
```


一个抽象方法，说明在GenericServlet类中并没有实现该内容，那么我们想到的是，在它上面肯定还有一层，也就是还有一个子类继承它，实现该方法，要是让我们自己写的Servlet继承GenericServlet，需要自己写service方法，那岂不是累死，并且我们可以看到，service方法中的参数还是ServletRequest，ServletResponse。并没有跟http相关对象挂钩，所以我们接着往下面看。

### HttpServlet类详解

继承了GenericServlet类，通过我们上面的推测，这个类主要的功能肯定是实现service方法的各种细节和设计。并且通过类名可以知道，该类就跟http挂钩了。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216152220082-1786372762.png)

关注service(HttpServletRequest req, HttpServletResponse resp)方法和service(ServletRequest req, ServletResponse res)方法。

其中 service(ServletRequest req, ServletResponse res)方法

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216152409566-1602415910.png)　　

该方法中就做一件事情，就是将ServletRequest和ServletResponse这两个对象强转为HttpServletRequest和HttpServletResponse对象。为什么能这样转？

首先要知道req、res是什么类型，通过打印System.out.println(req)，可以知道，req实际上的类型是org.apache.catalina.connector.RequestFacade　,Tomcat 的源码；
`public class RequestFacade implements HttpServletRequest`

`public interface HttpServletRequest extends ServletRequest`

通过图可以得知，req的继承结构：RequestFacade、httpServletRequest、ServletRequest，我们知道本身req是ServletRequest，那么从继承结构上看，它也可以看成HttpServletRequest，也可以看成ServletRequest，所以强转为HttpServletRequest是可以的，如果不明白，我举个例子，ArrayList、List、Object 这个，Object obj = new ArrayList();  List list = new ArrayList();  一个ArrayList对象可以看成List对象， 也可以看成一个Object对象，现在obj是不是可以堪称List对象呢？答案是可以的，因为obj就是ArrayList对象，既然是ArrayList对象，那么就可以看成是List对象。一样的道理，RequestFacade 对应 ArrayList、httpServleRequest对应 List、 ServletRequest 对应 Object。

转换为httpServletRequest和HttpServletResponse对象之后，在调用service(HttpServletRequest req, HttpServletResponse resp)方法。

service(HttpServletRequest req, HttpServletResponse resp)

这个方法就是判断浏览器过来的请求方式是哪种，每种的处理方式不一样，我们常用的就是get，post，并且，我们处理的方式可能有很多的内容，所以，在该方法内会将get，post等其他5种请求方式提取出来，变成单个的方法，然后我们需要编写Servlet时，就可以直接重写doGet或者doPost方法就行了，而不是重写service方法，更加有针对性。所以这里就回到了我们上面编写Servlet时的情况，继承httpServlet，而只要重写两个方法，一个doGet，一个doPost，其实就是service方法会调用这两个方法中的一个(看请求方式)。所以也就解答了我们一开始提的问题3。　　

## 四、几个重点的对象。ServletConfig、ServletContext，request、response
讲解四大类，ServletConfig对象，ServletContext对象、request对象，response对象

### （一）ServletConfig对象

获取途径：getServletConfig(); 

功能：上面大概提及了一下，能得到四个东西，

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216193829863-177507047.png)

* getServletName();  //获取Servlet的名称，也就是我们在web.xml中配置的Servlet-name
* getServletContext(); //获取ServletContext对象，该对象的作用看下面讲解
* getInitParameter(String); //获取在Servlet中初始化参数的值。这里注意与全局初始化参数的区分。这个获取的只是在该Servlet下的初始化参数

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170216195140550-371826071.png)

- getInitParameterNames(); //获取在Servlet中所有初始化参数的名字，也就是key值，可以通过key值，来找到各个初始化参数的value值。注意返回的是枚举类型

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217091747144-810654839.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217091914597-1623014325.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217091935300-1981582648.png)

注意：在上面我们所分析的源码过程中，我们就知道，其实可以不用先获得ServletConfig，然后在获取其各种参数，可以直接使用其方法，比如上面我们用的ServletConfig().getServletName();可以直接写成getServletName();而不用在先获取ServletConfig();了，原因就是在GenericServlet中，已经帮我们获取了这些数据，我们只需要直接拿就行。

ServletContext对象

获取途径：getServletContext(); 、getServletConfig().getServletContext();　　//这两种获取方式的区别就跟上面的解释一样，第一种是直接拿，在GenericServlet中已经帮我们用getServletConfig().getServletContext();拿到了ServletContext。我们只需要直接获取就行了，第二种就相当于我们自己在获取一遍，两种读是一样的。

功能： Tomcat为每个web项目都创建一个ServletContext实例， Tomcat在启动时创建，服务器关闭时销毁，在一个web项目中共享数据，管理web项目资源，为整个web配置公共信息等，通俗点讲，就是一个web项目，就存在一个ServletContext实例，每个Servlet读可以访问到它。

1、web项目中共享数据，getAttribute(String name)、setAttribute(String name, Object obj)、removeAttribute(String name)

setAttribute(String name, Object obj) 在web项目范围内存放内容，以便让在web项目中所有的Servlet读能访问到

getAttribute(String name) 通过指定名称获得内容

removeAttribute(String name) 通过指定名称移除内容  　

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217100021019-1747366315.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217100049254-1938338032.png)　　

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217100106379-1506943167.png)　　

2、整个web项目初始化参数 //这个就是全局初始化参数，每个Servlet中都能获取到该初始化值

getInitPatameter(String name)　　//通过指定名称获取初始化值

getInitParameterNames()　　//获得枚举类型

web.xml 配置 整个web项目的初始化

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217101303988-623871756.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217101317597-1875411632.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217101334441-1632535355.png)

3、获取web项目资源

3.1获取web项目下指定资源的路径：getServletContext().getRealPath("/WEB-INF/web.xml")

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217102136347-1305457655.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217102149113-1528997169.png)

3.2获取web项目下指定资源的内容，返回的是字节输入流。InputStream getResourceAsStream(java.lang.String path)

前提知识：需要了解流。不知道的可以去看看[IO流总结](http://www.cnblogs.com/whgk/p/5326568.html "http://www.cnblogs.com/whgk/p/5326568.html")的文章

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217105752019-1686962725.png)

输出内容截图一部分

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217105820394-1482656955.png)

4、getResourcePaths(java.lang.String path)  指定路径下的所有内容。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217110536316-752633624.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217110549457-1699480159.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217110604254-1174782701.png)

5还有很多别的方法，暂时用到的就这几个了，以后需要在用的，就查看源码，看API。

request对象

我们知道，request就是将请求文本封装而成的对象，所以通过request能获得请求文本中的所有内容，请求头、请求体、请求行 。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217113102004-1305201880.png)

1、请求行内容的获取。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217112157910-1493109100.png)　

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217112213488-1676592080.png)　

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217112234097-1109437412.png)　

2请求头的获取

随便百度一个东西，然后查看的请求头，包括以下这些内容，稍作了解。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217113256660-1823637829.png)

String getHeader(java.lang.String name) 获得指定头内容String【】

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217113439050-289451177.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217113449019-979867352.png)

long getDateHeader(java.lang.String name) 获得指定头内容Date

int getIntHeader(java.lang.String name)  获得指定头内容int

Enumeration getHeaders(java.lang.String name) 获得指定名称所有内容

3请求体的获取 -- 请求参数的获取

分两种，一种get请求，一种post请求

get请求参数：http://localhost:8080/test01/MyServlet?username=jack&password=1234

post请求参数: <form method="post"><input type="text" name="username">

String request.getParameter(String) 获得指定名称，一个请求参数值。

String[] request.getParameterValues(String) 获得指定名称，所有请求参数值。例如：checkbox、select等

Map<String , String[]> request.getParameterMap() 获得所有的请求参数　　

4请求转发

request.getRequestDispatcher(String path).forward(request,response);　　//path:转发后跳转的页面，这里不管用不用"/"开头，都是以web项目根开始，因为这是请求转发，请求转发只局限与在同一个web项目下使用，所以这里一直都是从web项目根下开始的，

web项目根：

开发：G:\Workspaces\test01\WebRoot\..

运行时：D:\java\ Tomcat\apache-tomcat-7.0.53\webapps\test01\..

web站点根：

运行时：D:\java\tomcat\apache-tomcat-7.0.53\webapps\..

从这里可以看出，web项目根就是从该web项目名开始，所以我们请求转发时，只需要接着项目名后面需要访问的路径写就行了，

特点：浏览器中url不会改变，也就是浏览器不知道服务器做了什么，是服务器帮我们跳转页面的，并且在转发后的页面，能够继续使用原先的request，因为是原先的request，所以request域中的属性都可以继续获取到。

response对象

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217141731800-1190823561.png)

常用的一个方法：response.setHeader(java.lang.String name, java.lang.String value) 设置指定的头，一般常用。

例如：设置每隔3秒就自动刷新一次，

response.setHeader("Refresh",3);

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217144321504-317554648.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217144436769-1330637125.png)![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170217144447535-1374034061.png)

这样可以看到现在时间的秒数，会发现每隔三秒就会自动刷新一次页面。

这个最重要的一个就是重定向，其他的一些操作都被封装到response对象中了，重点讲解重定向

重定向(页面跳转)

方式一：手动方案

response.setStatus(302);　　//状态码302就代表重定向

response.setHeader("location","http://www.baidu.com");

方式二：使用封装好的，通过response.sendRedirect("http://www.baidu.com");

特点：服务器告诉浏览器要跳转的页面，是浏览器主动去跳转的页面，浏览器知道，也浏览器的地址栏中url会变，是浏览器重新发起一个请求到另外一个页面，所以request是重新发起的，跟请求转发不一样。

注意：response.sendRedirect(path);　　//

第一种：response.sendRedirect("/test01/MyServlet01");　　//使用了"/"开头，说明是从web站点根开始，所以需要写test01/MyServlet01

第二种：response.sendRedirect("MyServlet01");　　//没有使用"/"开头，说明是从web项目根开始，那么就无需写test01了。

重定向没有任何局限，可以重定向web项目内的任何路径，也可以访问别的web项目中的路径，并且这里就用"/"区分开来，如果使用了"/"开头，就说明我要重新开始定位了，不访问刚才的web项目，自己写项目名，如果没有使用"/"开始，那么就知道是访问刚才那个web项目下的Servlet，就可以省略项目名了。就是这样来区别。

五、总结

这一章节篇幅较长，不过理清很多知识点

1、什么是Servlet？如果编写Servlet？

2、分析了Servlet的部分源码，知道了其中的一些设计巧妙的东西，比如，本来编写Servlet是能看到其生命周期的，但是在其设计下，我们只关注doGet和doPost方法，为什么能这样呢？就可以通过源码中得知。

3、Servlet的生命周期，web.xml的配置

4、Servlet中的ServletConfig对象，ServletContext对象，request对象，response对象的详细讲解。包括其中的一些常用的方法。

5、下一篇讲解一下request、response的中文乱码问题的解决