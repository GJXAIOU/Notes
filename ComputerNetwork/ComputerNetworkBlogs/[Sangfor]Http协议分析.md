# Http协议分析


目录：
- HTTP请求和响应
- 状态和回话
- cookie欺骗
- Burp suit


## 一、HTTP协议解析

HTTP（HyperText Transfer Protocol）即超文本传输协议，是一种详细规定了浏览器和万维网服务器之间互相通信的规则，它是万维网交换信息的基础，它允许**将HTML（超文本标记语言）文档从Web服务器传送到Web浏览器**。

HTTP协议目前最新版的版本是1.1，HTTP是一种无状态的协议，**无状态是指Web浏览器与Web服务器之间不需要建立持久的连接**，这意味着当一个客户端向服务器端发出请求，然后Web服务器返回响应（Response），连接就被关闭了，在服务器端不保留连接的有关信息。也就是说，**HTTP请求只能由客户端发起**，而服务器不能主动向客户端发送数据。

HTTP遵循请求（Request）/应答（Response）模型，Web浏览器向Web服务器发送请求时，Web服务器处理请求并返回适当的应答，如图所示。

HTTP使用一种**基于消息的模型**：客户端发送一条请求消息，而后由服务器返回一条响应消息。


### (一)HTTP请求字段
```language
•POST /test.php HTTP/1.1//请求行

•HOST：www.test.com //请求头

•User-Agent：Mozilla/5.0 （windows NT 6.1；rv：15.0）Gecko/20100101 Firefox/15.0 //空白行，代表请求头结束

•Username=admin&password=admin //请求正文
```

HTTP请求包括三部分，分别是请求行（请求方法）、请求头（消息报头）和请求正文。

HTTP请求第一行为请求行，由三部分组成，第一部分说明了该请求时POST请求，第二部分是**一个斜杠**（/login.php），**用来说明请求是该域名根目录下的login.php**，第三部分说明使用的是HTTP1.1版本。

HTTP请求第二行至空白行为请求头（也被称为消息头）。其中，**HOST代表该客户机所请求主机地址**，User-Agent代表浏览器的标识，请求头由客户端自行设定。

HTTP请求第三行为请求正文，**请求正文是可选的**，它最常出现在POST请求方式中。

### （二）HTTP请求方法
GET：**GET方法用于获取请求页面的指定信息**。如果**请求资源为动态脚本（非HTML），那么返回文本是Web容器解析后的HTML源代码**。GET请求**没有消息主体**，因此在消息头后的空白行是没有其他数据。

POST：POST方法也与GET方法相似，但最大的区别在于，GET方法没有请求内容，而**POST是有请求内容的**。

HEAD：这个请求的功能与GET请求相似，不同之处在于服务器不会再其响应中返回消息主体，因此，这种方法可**用于检查某一资源在向其提交GET请求前是否存在**。

PUT：PUT方法用于**请求服务器把请求中的实体存储在请求资源下**，如果请求资源已经在服务器中存在，那么将会用此请求中的数据**替换**原先的数据。向服务器上传指定的资源。




### （三）HTTP响应

```
•HTTP/1.1 200 OK //响应行
•Date: Sun, 15 Nov 2015 11:02:04 GMT //响应头
•Server: bfe/1.0.8.9
•Content-Length: 2605
•Content-Type: application/javascript
•Cache-Control: max-age=315360000
•Expires: Fri, 13 Jun 2025 09:54:00 GMT
•Content-Encoding: gzip
•Set-Cookie: H_PS_PSSID=2022_1438_1944_1788; path=/; domain=test.com
•Connection: keep-alive
• //空白行，代表响应头结束
•<html>
•<head><title> Index.html </title></head> //响应正文消息主题
```

HTTP响应的第一行为响应行，其中有HTTP版本（HTTP/1.1）、状态码（200）以及**消息**“OK”。

第二行至末尾的空白行为响应头，由服务器向客户端发送。

消息头之后是**响应正文，是服务器向客户端发送的HTML数据**。

- HTTP状态码

  - 五种状态码：
    - 1xx：信息提示，表示请求已被**成功接收**，继续处理。
    - 2xx：请求被**成功提交**。但不能说明被接收
    - 3xx：客户端被**重定向**到其他资源。
    - 4xx：客户端错误状态码，格式错误或者不存在资源。
    - 5xx：描述服务器内部错误。

  - 常见的状态码描述如下：
      - 200：客户端请求成功，是最常见的状态。
      - 302：重定向。
      - 404：**请求资源不存在**，是最常见的状态。
      - 400：客户端请求有语法错误，不能被服务器所理解。
      - 401：请求未经授权。
      - 403：服务器收到请求，但是**拒绝提供服务**。
      - 500：服务器内部错误，是最常见的状态。
      - 503：服务器当前不能处理客户端的请求。



### (四 )HTTP消息头
1.请求头：**请求头只出现在HTTP请求中**，请求报头允许客户端向服务端传递请求的附加信息和客户端自身信息。

2.响应头：响应头是服务器根据请求向客户端发送的HTTP头。


- **常见的请求头**：

```
•Host
•User-Agent
•Referer
•Cookie
•Accept
```
①Host 请求报头域主要用于**指定被请求资源的Internet主机和端口**。
②User-Agent 请求报头域允许客户端将它的操作系统、浏览器和其他属性告诉服务器
③Referer  包含一个URL，**代表当前访问URL的上一个URL**，也就是说，用户是从什么地方来到本页面。当前请求的原始URL地址。
④Cookie 是非常重要的请求头，常用来**表示请求者的身份**等。
⑤Accept 这个消息头用于告诉服务器**客户端愿意接受那些内容**，比如图像类，办公文档格式等等。

- **常见的响应头**

```
•Server
•Location
•Content-Type
•Content-Encoding
•Content-Length
•Connection
```
①Server 服务器使用的Web服务器名称。

②Location **服务器通过这个头告诉浏览器去访问哪个页面**，浏览器接收到这个请求之后，通常会立刻访问Location头所指向的页面。**用于在重定向响应中说明重定向的目标地址**。

③Content-Type 这个消息头用于**规定主体的内容类型**。例如，HTML文档的内容类型text/html。

④Content-Encoding 这个消息头为消息主体中的内容**指定编码形式**，一些应用程序使用它来**压缩响应以加快传输速度**。

⑤Content-Length 消息头规定**消息主体的字节长度**。实体头用于指明实体正文的长度，以字节方式存储的**十进制**数字来表示。

⑥Connection 允许发送**指定连接的选项**。

![拦截HTTP请求的分析点]($resource/%E6%8B%A6%E6%88%AAHTTP%E8%AF%B7%E6%B1%82%E7%9A%84%E5%88%86%E6%9E%90%E7%82%B9.png)



## 二、会话和会话状态简介 -- cookie

•WEB应用中的会话是指一个客户端浏览器与WEB服务器之间**连续发生的一系列请求和响应过程**。

•WEB应用的会话状态是指WEB服务器与浏览器在会话过程中产生的**状态信息**，借助会话状态，WEB服务器能够**把属于同一会话中的一系列的请求和响应过程关联起来**。

### （一）如何实现有状态的会话
•某个用户从网站的登录页面登入后，再进入购物页面购物时，负责处理购物请求的服务器程序必须知道处理上一次请求的程序所得到的用户信息。

•HTTP协议是一种无状态的协议，**WEB服务器本身不能识别出哪些请求是同一个浏览器发出的** ，浏览器的每一次请求都是完全孤立的。

•WEB服务器端程序要能从大量的请求消息中区分出哪些请求消息属于同一个会话，即能识别出来自同一个浏览器的访问请求，这需要浏览器对其发出的每个请求消息都进行标识，**属于同一个会话中的请求消息都附带同样的标识号**，而属于不同会话的请求消息总是附带不同的标识号，这个标识号就**称之为会话ID（SessionID）**。

•**会话ID可以通过一种称之为Cookie的技术在请求消息中进行传递，也可以作为请求URL的附加参数进行传递**。会话ID是WEB服务器为每客户端浏览器分配的一个唯一代号，它通常是在WEB服务器接收到某个浏览器的第一次访问时产生，并且随同响应消息一道发送给浏览器。

•**会话过程由WEB服务器端的程序开启**，一旦开启了一个会话，服务器端程序就要为这个会话创建一个独立的存储结构来保存该会话的状态信息，同一个会话中的访问请求都可以且只能访问属于该会话的存储结构中的状态信息。


### （二）cookie
[详解见这里：cookie详解](cookie详解)

•Cookie是一种在**客户端**保持HTTP状态信息的技术，它好比商场发放的优惠卡。

•Cookie是在浏览器访问WEB服务器的某个资源时，由WEB服务器在HTTP响应消息头中附带传送给浏览器的一片数据，WEB服务器传送给各个客户端浏览器的数据是可以各不相同的。

•一旦WEB浏览器保存了某个Cookie，那么它在以后每次访问该WEB服务器时，都应在HTTP请求头中将这个Cookie回传给WEB服务器。

 Cookie是一小段文本信息，伴随着用户请求和页 面在Web服务器和浏览器之间传递。Cookie包含每次 用户访问站点时Web应用程序都可以读取的信息。

**Cookie只是一段文本，所以它只能保存字符串**。

•WEB服务器通过在HTTP响应消息中增加Set-Cookie响应头字段将Cookie信息发送给浏览器，浏览器则通过在HTTP请求消息中增加Cookie请求头字段将Cookie回传给WEB服务器。

•一个Cookie只能标识一种信息，它**至少含有一个标识该信息的名称（NAME）和设置值（VALUE）**。

•**一个WEB站点可以给一个WEB浏览器发送多个Cookie，一个WEB浏览器也可以存储多个WEB站点提供的Cookie**。

•浏览器一般只允许存放300个Cookie，每个站点最多存放20个Cookie，每个Cookie的大小限制为4KB。

![cookie传送过程]($resource/cookie%E4%BC%A0%E9%80%81%E8%BF%87%E7%A8%8B.png)



### （三）cookie功能特点
•存储于浏览器头部/传输于HTTP头部
•写时带属性，读时无属性
•HTTP头中Cookie: user=bob; cart=books;
•属性 name/value/expire/domain/path/httponly/secure/......
•由三元组[name,domain,path]唯一确定cookie
•唯一确定！= cookie唯一


### （四）set-cookie2响应头字段

•Set-Cookie2头字段用于指定WEB服务器向客户端传送的Cookie内容，但是按照Netscape规范实现Cookie功能的WEB服务器，使用的是Set-Cookie头字段，两者的语法和作用类似。

•Set-Cookie2头字段中设置的cookie内容是具有一定格式的字符串，它必须以Cookie的名称和设置值开头，格式为“**名称=值**”，后面可以加上0个或多个以分号（;）和空格分隔的其它可选属性，属性格式一般为“属性名=值”。

Ø举例：Set-Cookie2: user=hello; Version=1; Path=/
•除了“名称=值”对必须位于最前面外，其它的可选属性的先后顺序可以任意。

•Cookie的名称只能由普通的英文ASCII字符组成，浏览器不用关心和理解Cookie的值部分的意义和格式，只要WEB服务器能理解值部分的意义就行。

•大多数现有的WEB服务器都是采用某种编码方式将值部分的内容编码成可打印的ASCII字符，RFC 2965规范中没有明确限定编码方式。


- 响应头字段中的属性

```language
•Comment=value
•Discard
•Domain=value

```

Ø例如：Set-Cookie2: user=hello; Version=1; Path=/; Domain=.hello.org

Ø**Domain 属性定义可访问该cookie的域名**，对一些大的网站，如果希望cookie可以在子网站中共享，可以使用该属性。例如设置Domain为 .bigsite.com ,则sub1.bigsite.com和sub2.bigsite.com都可以访问已保存在客户端的cookie，这时还需要将Path设置为/。

•Max-Age=value  定义cookie的有效时间，以秒计算，当超过有效期后，cookie的信息不会从客户端附加在HTTP消息头中发送到服务端。

•Path=value定义网站上可以访问cookie的页面的路径，缺省状态下Path为产生cookie时的路径，此时cookie可以被该路径以及其子路径下的页面访问；**可以将Path设置为/，使cookie可以被网站下所有页面访问**。

•Port[="portlist"]

•Secure属性值定义cookie的安全性，当该值为true时必须是HTTPS状态下cookie才从客户端附加在HTTP消息中发送到服务端

•Version=value定义cookie的版本，由cookie的创建者定义。


### （五）cookie请求头字段

- Cookie请求头字段中的每个Cookie之间用逗号（,）或分号（;）分隔。

- 在Cookie请求头字段中除了必须有“名称=值”的设置外，还可以有Version、Path、Domain、Port等几个属性。

- 在Version、Path、Domain、Port等属性名之前，都要**增加一个“$”字符作为前缀**。

- **Version属性只能出现一次，且要位于Cookie请求头字段设置值的最前面，如果需要设置某个Cookie信息的 Path、Domain、Port等属性，它们必须位于该Cookie信息的“名称=值”设置之后**。

* 浏览器使用Cookie请求头字段将Cookie信息回送给WEB服务器。
* 多个Cookie信息通过一个Cookie请求头字段回送给WEB服务器。 
* 浏览器根据下面的几个规则决定是否发送某个Cookie信息：

  * 请求的主机名是否与某个存储的Cookie的Domain属性匹配；
  * 请求的端口号是否在该Cookie的Port属性列表中；
  * 请求的资源路径是否在该Cookie的Path属性指定的目录及子目录中；
  * 该Cookie的有效期是否已过。

- Path属性指向**子目录**的Cookie排**在**Path属性指向**父目录的Cookie之前**。   
举例：Cookie: $Version=1; Course=Java; $Path=/hello/lesson; Course=vc; $Path=/hello


### （六）cookie的安全属性

- secure属性

  - 当设置为**true**时，表示创建的 Cookie 会被以安全的形式向服务器传输，也就是**只能在 HTTPS 连接中被浏览器传递到服务器端进行会话验证**，如果是 HTTP 连接则不会传递该信息，所以不会被窃取到Cookie 的具体内容。

- HttpOnly属性

  * 如果在Cookie中**设置了"HttpOnly"属性**，那么**通过程序(JS脚本、Applet等)将无法读取到Cookie信息，这样能有效的防止XSS攻击**。
  * secure属性是防止信息在**传递的过程中**被监听捕获后信息泄漏，HttpOnly属性的目的是**防止程序获取**cookie后进行攻击。
  * 这两个属性**并不能解决**cookie在本机出现的**信息泄漏的问题**(FireFox的插件FireBug能直接看到cookie的相关信息)。


## 三、回话与回话状态简介 -- session 

•使用**Cookie和附加URL参数**都可以将上一次请求的状态信息传递到下一次请求中，但是**如果传递的状态信息较多，将极大降低网络传输效率和增大服务器端程序处理的难度**。

•**Session技术是一种将会话状态保存在服务器端的技术** ，它可以比喻成是医院发放给病人的病历卡和医院为每个病人保留的病历档案的结合方式 。

•客户端需要接收、记忆和回送 Session的会话标识号，Session可以且通常是借助Cookie来传递会话标识号。


### （一）session的跟踪机制

•Servlet API规范中定义了一个HttpSession接口，HttpSession接口定义了各种管理和操作会话状态的方法。

•HttpSession对象是保持会话状态信息的存储结构，一个客户端在WEB服务器端对应一个各自的HttpSession对象。

•WEB服务器并不会在客户端开始访问它时就创建HttpSession对象，**只有客户端访问某个能与客户端开启会话的Servlet程序时，WEB应用程序才会创建一个与该客户端对应的HttpSession对象**。

•WEB服务器为HttpSession对象分配一个独一无二的会话标识号，然后在响应消息中将这个**会话标识号**传递给客户端。客户端需要记住会话标识号，并在后续的每次访问请求中都把这个会话标识号传送给WEB服务器，WEB服务器端程序依据回传的会话标识号就知道这次请求是哪个客户端发出的，从而选择与之对应的HttpSession对象。

•WEB应用程序创建了与某个客户端对应的HttpSession对象后，只要**没有超出一个限定的空闲时间段，HttpSession对象就驻留在WEB服务器内存之中，该客户端此后访问任意的Servlet程序时，它们都使用与客户端对应的那个已存在的HttpSession对象**。

•HttpSession接口中专门定义了一个setAttribute方法来将对象存储到HttpSession对象中，还定义了一个getAttribute方法来检索存储在HttpSession对象中的对象，存储进HttpSession对象中的对象可以被属于同一个会话的各个请求的处理程序共享。

•Session是实现网上商城的购物车的最佳方案，存储在某个客户Session中的一个集合对象就可充当该客户的一个购物车。



### （二）session的超时管理


•WEB服务器无法判断当前的客户端浏览器是否还会继续访问，也无法检测客户端浏览器是否关闭，所以，即使客户已经离开或关闭了浏览器，WEB服务器还要保留与之对应的HttpSession对象。

•随着时间的推移而不断增加新的访问客户端，WEB服务器内存中将会因此积累起大量的不再被使用的HttpSession对象，并将最终导致服务器内存耗尽。

•WEB服务器采用“超时限制”的办法来判断客户端是否还在继续访问，如果某个客户端在一定的时间之内没有发出后续请求，WEB服务器则认为客户端已经停止了活动，结束与该客户端的会话并将与之对应的HttpSession对象变成垃圾。

•如果客户端浏览器**超时后再次发出访问请求，WEB服务器则认为这是一个新的会话的开始**，将为之创建新的HttpSession对象和分配新的会话标识号。

•会话的超时间隔可以在**web.xml文件中设置，其默认值由Servlet容器定义**。

•  <session-config>

• <session-timeout>30</session-timeout>

• </session-config>



### （三）利用cookie实现session跟踪

•如果WEB服务器处理某个访问请求时创建了新的HttpSession对象，它将把会话标识号作为一个Cookie项加入到响应消息中，通常情况下，浏览器在随后发出的访问请求中又将会话标识号以Cookie的形式回传给WEB服务器。

•WEB服务器端程序依据回传的会话标识号就知道以前已经为该客户端创建了HttpSession对象，不必再为该客户端创建新的HttpSession对象，而是直接使用与该会话标识号匹配的HttpSession对象，通过这种方式就实现了对同一个客户端的会话状态的跟踪。


### （四）利用URL重写实现Session跟踪

- Servlet规范中引入了**一种补充的会话管理机制**，它**允许不支持Cookie的浏览器也可以与WEB服务器保持连续的会话。这种补充机制要求在响应消息的实体内容中必须包含下一次请求的超链接，并将会话标识号作为超链接的URL地址的一个特殊参数**。

 - 将会话标识号以参数形式**附加在超链接的URL地址后面的技术称为URL重写**。如果在浏览器不支持Cookie或者关闭了Cookie功能的情况下，WEB服务器还要能够与浏览器实现有状态的会话，就必须对所有可能被客户端访问的请求路径（包括超链接、form表单的action属性设置和重定向的URL）进行URL重写。


- HttpServletResponse接口中定义了两个用于完成URL重写方法：
  - encodeURL方法
  - encodeRedirectURL方法



## 四、cookie和session的不同


•session和cookies同样都是针对单独用户的变量（或者说是对象好像更合适点），不同的用户在访问网站的时候 都会拥有各自的session或者cookies，不同用户之间互不干扰。

•他们的不同点是：

•1，**存储位置不同**

- **session在服务器端产生**，比较安全，但是如果session较多则会影响性能

- **cookies在客户端产生**，安全性稍弱

•2，生命周期不同

- session生命周期 在指定的时间（如20分钟）到了之后会结束，不到指定的时间，也会随着浏览器进程的结束而结束。

- cookies默认情况下也随着浏览器进程结束而结束，但**如果手动指定时间，则不受浏览器进程结束的影响**。 

•