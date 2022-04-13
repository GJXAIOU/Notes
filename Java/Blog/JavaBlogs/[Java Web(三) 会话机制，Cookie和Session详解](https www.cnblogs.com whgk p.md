# [Java Web(三) 会话机制，Cookie和Session详解](https://www.cnblogs.com/whgk/p/6422391.html)

![session流程]($resource/session%E6%B5%81%E7%A8%8B.png)

很大一部分应该知道什么是会话机制，也能说的出几句，我也大概了解一点，但是学了之后几天不用，立马忘的一干二净，原因可能是没能好好理解这两种会话机制，所以会一直遗忘，一直重新回过头来学习它，今天好好把他总结一下，[借鉴该文章中的内容](http://blog.csdn.net/keda8997110/article/details/16922815 "http://blog.csdn.net/keda8997110/article/details/16922815")，因为我觉得该篇文章确实写的很不错，解答了我很多疑问，特点是对cookie和session的理解，其中的会员卡的例子，真是一针见血的奇效。我按照自己的思路来重新整理一份，给自己以后看。

--WZY

一、会话机制

Web程序中常用的技术，用来**跟踪用户的整个会话**。常用的会话跟踪技术是Cookie与Session。**Cookie通过在客户端记录信息确定用户身份**，**Session通过在服务器端记录信息确定用户身份**。

一次会话指的是：就好比打电话，A给B打电话，接通之后，会话开始，直到挂断电话，该次会话就结束了，而浏览器访问服务器，就跟打电话一样，浏览器A给服务器发送请求，访问web程序，该次会话就已经接通，其中不管浏览器发送多少请求(就相当于接通电话后说话一样)，都视为一次会话，直到浏览器关闭，本次会话结束。其中注意，一个浏览器就相当于一部电话，如果使用火狐浏览器，访问服务器，就是一次会话了，然后打开google浏览器，访问服务器，这是另一个会话，虽然是在同一台电脑，同一个用户在访问，但是，这是两次不同的会话。

知道了什么是会话后，思考一个问题，一个浏览器访问一个服务器就能建立一个会话，如果别的电脑，都同时访问该服务器，就会创建很多会话，就拿一些购物网站来说，我们访问一个购物网站的服务器，会话就被创建了，然后就点击浏览商品，对感兴趣的商品就先加入购物车，等待一起付账，这看起来是很普通的操作，但是想一下，如果有很多别的电脑上的浏览器同时也在访问该购物网站的服务器，跟我们做类似的操作呢？服务器又是怎么记住用户，怎么知道用户A购买的任何商品都应该放在A的购物车内，不论是用户A什么时间购买的，不能放入用户B或用户C的购物车内的呢？所以就有了cookie和session这两个技术，就像第一行说的那样，cookie和session用来跟踪用户的整个会话，

Cookie和Session之间的区别和联系

假如一个咖啡店有喝5杯咖啡免费赠一杯咖啡的优惠，然而一次性消费5杯咖啡的机会微乎其微，这时就需要某种方式来纪录某位顾客的消费数量。想象一下其实也无外乎下面的几种方案：

1、该店的店员很厉害，能记住每位顾客的消费数量，只要顾客一走进咖啡店，店员就知道该怎么对待了。这种做法就是协议本身支持状态。但是http协议本身是无状态的

2、发给顾客一张卡片，上面记录着消费的数量，一般还有个有效期限。每次消费时，如果顾客出示这张卡片，则此次消费就会与以前或以后的消费相联系起来。这种做法就是在客户端保持状态。也就是cookie。 顾客就相当于浏览器，cookie如何工作，下面会详细讲解

3、发给顾客一张会员卡，除了卡号之外什么信息也不纪录，每次消费时，如果顾客出示该卡片，则店员在店里的纪录本上找到这个卡号对应的纪录添加一些消费信息。这种做法就是在服务器端保持状态。

由于HTTP协议是无状态的，而出于种种考虑也不希望使之成为有状态的，因此，后面两种方案就成为现实的选择。具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的，但实际上它还有其他选择

二、Cookie

上面已经介绍了为什么要使用Cookie，以及Cookie的一些特点，比如保存在客户端，用来记录用户身份信息的，现在来看看如何使用Cookie。

借着上面会员卡的例子来说，采用的是第二种方案，其中还需要解决的问题就是：如何分发会员卡，会员卡的内容，客户如何使用会员卡，会员卡的有效日期，会员卡的使用范围

1、如何分发会员卡、会员卡的内容：也就是cookie是如何创建的？创建后如何发送给客户端？

由服务器进行创建，也就相当于咖啡店来创建会员卡，在创建会员卡的同时，就会将会员卡中的内容也给设置了

Cookie cookie = new Cookie(key,value);　　//以键值对的方式存放内容，

response.addCookie(cookie);　　//发送回浏览器端

注意：一旦cookie创建好了，就不能在往其中增加别的键值对，但是可以修改其中的内容，

cookie.setValue();　　//将key对应的value值修改

2、客户如何使用会员卡，cookie在客户端是如何工作的，工作原理是什么？

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170221095538616-1829815211.png)

这个过程就相当于，咖啡店创建好了会员卡，并且已经设置了其中的内容，交到了客户手中，下次客户过来时，就带着会员卡过来，就知道你是会员了，然后咖啡店就拿到你的会员卡对其进行操作。

3、会员卡的有效日期？也就是cookie也是拥有有效日期的。

这个可以自由设置，默认是关闭浏览器，cookie就没用了。

cookie.setMaxAge(expiry);　　//设置cookie被浏览器保存的时间。

expiry：单位秒，默认为-1，

expiry=-1：代表浏览器关闭后，也就是会话结束后，cookie就失效了，也就没有了。

expiry>0：代表浏览器关闭后，cookie不会失效，仍然存在。并且会将cookie保存到硬盘中，直到设置时间过期才会被浏览器自动删除，

expiry=0：删除cookie。不管是之前的expiry=-1还是expiry>0，当设置expiry=0时，cookie都会被浏览器给删除。

4、会员卡的使用范围？比如星巴克在北京有一个分店，在上海也有一个分店，我们只是在北京的星巴克办理了会员卡，那么当我们到上海时，就不能使用该会员卡进行打折优惠了。而cookie也是如此，可以设置服务器端获取cookie的访问路径，而并非在服务器端的web项目中所有的servlet都能访问该cookie。

cookie默认路径：当前访问的servlet父路径。

例如：http://localhost:8080/test01/a/b/c/SendCookieServlet

默认路径：/test01/a/b/c　　也就是说，在该默认路径下的所有Servlet都能够获取到cookie，/test01/a/b/c/MyServlet　这个MyServlet就能获取到cookie。

修改cookie的访问路径

setPath("/")；　　//在该服务器下，任何项目，任何位置都能获取到cookie，

通途：保证在tomcat下所有的web项目可以共享相同的cookie 

例如：tieba , wenku , beike 多个项目共享数据。例如用户名。

setPath("/test01/");　　//在test01项目下任何位置都能获取到cookie。

5、总结Cookie：

工作流程：

1\. servlet创建cookie，保存少量数据，发送浏览器。

2\. 浏览器获得服务器发送的cookie数据，将自动的保存到浏览器端。

3\. 下次访问时，浏览器将自动携带cookie数据发送给服务器。

cookie操作

1.创建cookie：new Cookie(name,value)

2.发送cookie到浏览器：HttpServletResponse.addCookie(Cookie)

3.servlet接收cookie：HttpServletRequest.getCookies()  浏览器发送的所有cookie

cookie特点

1\. 每一个cookie文件大小：4kb ， 如果超过4kb浏览器不识别

2\. 一个web站点（web项目）：发送20个

3.一个浏览器保存总大小：300个

4.cookie 不安全，可能泄露用户信息。浏览器支持禁用cookie操作。

5\. 默认情况生命周期：与浏览器会话一样，当浏览器关闭时cookie销毁的。---临时cookie

cookie api

getName() 获得名称，cookie中的key

getValue() 获得值，cookie中的value

setValue(java.lang.String newValue)  设置内容，用于修改key对应的value值。

setMaxAge(int expiry) 设置有效时间【】

setPath(java.lang.String uri)  设置路径【】　　

setDomain(java.lang.String pattern) 设置域名 , 一般无效，有浏览器自动设置，setDomain(".itheima.com")

www.itheima.com / bbs.itheima.com 都可以访问

a.b.itheima.com无法访问

作用：设置cookie的作用范围，域名+路径在一起就构成了cookie的作用范围，上面单独设置的setPath有用，是因为有浏览器自动设置该域名属性，但是我们必须知道有这么个属性进行域名设置的

isHttpOnly()  是否只是http协议使用。只能servlet的通过getCookies()获得，javascript不能获得。

setComment(java.lang.String purpose) (了解)　　//对该cookie进行描述的信息(说明作用)，浏览器显示cookie信息时能看到

setSecure(boolean flag) (了解)　　是否使用安全传输协议。为true时，只有当是https请求连接时cookie才会发送给服务器端，而http时不会，但是服务端还是可以发送给浏览端的。

setVersion(int v) (了解)　　参数为0（传统Netscape cookie规范编译）或1（RFC 2109规范编译）。这个没用到，不是很懂

注意：cookie不能发送中文，如果要发送中文，就需要进行特别处理。

JDK提供工具，进行编码

URLEncoder:编码

URLDecoder:解码

//发送cookie

Cookie cookie = new Cookie(URLEncoder.encode("哈哈"),URLEncoder.encode("呵呵"));

response.addCookie(cookie);

//获得cookie中文内容

URLDecoder.decoder(request.getCookie().getName);　　//获取key

URLDecoder.decoder(request.getCookie().getValue);　　//获取value　　

6.cookie案例

6.1、记住用户名

登录时，在服务器端获取到用户名，然后创建一个cookie，将用户名存入cookie中，发送回浏览器端，然后浏览器下次在访问登录页面时，先拿到cookie，将cookie中的信息拿出来，看是否保存了该用户名，如果保存了，那么直接用他，如果没有，则自己手写用户名。

6.2、历史记录

比如购物网站，都会有我们的浏览记录的，实现原理其实也是用cookie技术，每浏览一个商品，就将其存入cookie中，到需要显示浏览记录时，只需要将cookie拿出来遍历即可。　　

三、Session

同样，会员卡的例子的第三种方法，发给顾客一张会员卡，除了卡号之外什么信息也不纪录，每次消费时，如果顾客出示该卡片，则店员在店里的纪录本上找到这个卡号对应的纪录添加一些消费信息。这种做法就是在服务器端保持状态。　这就是session的用法，在服务器端来保持状态，保存一些用户信息。

功能作用：服务器用于共享数据技术，

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170221144715007-894494637.png)　

session原理分析：

首先浏览器请求服务器访问web站点时，程序需要为客户端的请求创建一个session的时候，服务器首先会检查这个客户端请求是否已经包含了一个session标识、称为SESSIONID，如果已经包含了一个sessionid则说明以前已经为此客户端创建过session，服务器就按照sessionid把这个session检索出来使用，如果客户端请求不包含session id，则服务器为此客户端创建一个session并且生成一个与此session相关联的session id，sessionid 的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个sessionid将在本次响应中返回到客户端保存，保存这个sessionid的方式就可以是cookie，这样在交互的过程中，浏览器可以自动的按照规则把这个标识发回给服务器，服务器根据这个sessionid就可以找得到对应的session，又回到了这段文字的开始。

获取session：

request.getSession();　　//如果没有将创建一个新的，等效getSession(true);

有些人不理解，为什么是通过request来获取session，可以这样理解，在获取session时，需要检测请求中是否有session标识，所以需要用request来获取

request.getSession(boolean);　　//true：没有将创建，false：没有将返回null

session属性操作：

xxxAttribute(...)

用来存放一些信息，然后才能共享信息　

setAttrubute(key,value);

getAttribute(key);

session生命周期

常常听到这样一种误解“只要关闭浏览器，session就消失了”。其实可以想象一下会员卡的例子，除非顾客主动对店家提出销卡，否则店家绝对不会轻易删除顾客的资料。对session来说也是一样的，除非程序通知服务器删除一个session，否则服务器会一直保留，程序一般都是在用户做log off的时候发个指令去删除session。然而浏览器从来不会主动在关闭之前通知服务器它将要关闭，因此服务器根本不会有机会知道浏览器已经关闭，之所以会有这种错觉，是大部分session机制都使用会话cookie来保存session id，而关闭浏览器后这个session id就消失了，再次连接服务器时也就无法找到原来的session。如果服务器设置的cookie被保存到硬盘上，或者使用某种手段改写浏览器发出的HTTP请求头，把原来的session id发送给服务器，则再次打开浏览器仍然能够找到原来的session　

恰恰是由于关闭浏览器不会导致session被删除，迫使服务器为seesion设置了一个失效时间，一般是30分钟，当距离客户端上一次使用session的时间超过这个失效时间时，服务器就可以认为客户端已经停止了活动，才会把session删除以节省存储空间

我们也可以自己来控制session的有效时间

session.invalidate()将session对象销毁

setMaxInactiveInterval(int interval) 设置有效时间，单位秒

在web.xml中配置session的有效时间

<session-config>

<session-timeout>30</session-timeout>   单位：分钟

<session-config>

所以，讨论了这么就，session的生命周期就是：

创建：第一次调用getSession()

销毁：

1、超时，默认30分钟

2、执行api：session.invalidate()将session对象销毁、setMaxInactiveInterval(int interval) 设置有效时间，单位秒

3、服务器非正常关闭

自杀，直接将JVM马上关闭

如果正常关闭，session就会被持久化(写入到文件中，因为session默认的超时时间为30分钟，正常关闭后，就会将session持久化，等30分钟后，就会被删除)

位置：　D:\java\tomcat\apache-tomcat-7.0.53\work\Catalina\localhost\test01\SESSIONS.ser

session id的URL重写

当浏览器将cookie禁用，基于cookie的session将不能正常工作，每次使用request.getSession() 都将创建一个新的session。达不到session共享数据的目的，但是我们知道原理，只需要将session id 传递给服务器session就可以正常工作的。

解决：通过URL将session id 传递给服务器：URL重写

手动方式： url;jsessionid=....

api方式：

encodeURL(java.lang.String url) 进行所有URL重写

encodeRedirectURL(java.lang.String url) 进行重定向 URL重写　

这两个用法基本一致,只不过考虑特殊情况,要访问的链接可能会被Redirect到其他servlet去进行处理,这样你用上述方法带来的session的id信息不能被同时传送到其他servlet.这时候用encodeRedirectURL()方法就可以了　

如果浏览器禁用cooke，api将自动追加session id ，如果没有禁用，api将不进行任何修改。

注意：如果浏览器禁用cookie，web项目的所有url都需进行重写。否则session将不能正常工作

当禁止了cookie时，

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170221191047757-1942818322.png)　　　　

四、总结

知道了什么是cookie和什么是session？

cookie是一种在客户端记录用户信息的技术，因为http协议是无状态的，为了解决这个问题而产生了cookie。记录用户名等一些应用

session是一种在服务端记录用户信息的技术，一般session用来在服务器端共享数据，

cookie的工作原理？session的工作原理？

cookie工作原理，可以看上面讲解cookie的那张图，cookie是由服务器端创建发送回浏览器端的，并且每次请求服务器都会将cookie带过去，以便服务器知道该用户是哪一个。其cookie中是使用键值对来存储信息的，并且一个cookie只能存储一个键值对。所以在获取cookie时，是会获取到所有的cookie，然后从其中遍历。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170221191714351-598342841.png)

session的工作原理就是依靠cookie来做支撑，第一次使用request.getSession()时session被创建，并且会为该session创建一个独一无二的sessionid存放到cookie中，然后发送会浏览器端，浏览器端每次请求时，都会带着这个sessionid，服务器就会认识该sessionid，知道了sessionid就找得到哪个session。以此来达到共享数据的目的。 这里需要注意的是，session不会随着浏览器的关闭而死亡，而是等待超时时间。

如果对cookie和session还有不理解的地方，用大家肯定都会用，就是需要理解，为什么需要使用cookie和session，可以看看那个会员卡的例子，cookie和session只是为了解决http协议无状态的这种缺陷，为了记录用户信息，记录浏览器和服务器之间的状态和衍生出来的。